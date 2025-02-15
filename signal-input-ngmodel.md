Recently, I encountered this challenge... Refactor a form component to [Angular Signals](https://angular.dev/guide/signals). 

The old form component works like this:

- The form data comes from a reactive state service 
- The form data is an **object**
- The form data is cloned before it is passed to the form component
- The form component receives the form data via one classic [decorator-based **Angular @Input**](https://angular.dev/guide/components/inputs#declaring-inputs-with-the-input-decorator)
```ts
@Input({required: true})
user!: User;
```
- The form uses **[[(ngModel)]](https://angular.dev/guide/forms/template-driven-forms#bind-input-controls-to-data-properties) to mutate the form data object**
```html
<div>
  <label for="firstName">First Name</label>
  <input id="firstName" name="firstName" [(ngModel)]="user.firstName" />
</div>
```
- Clicking the save button would send the mutated object to the parent component via an Angular @Output
- The parent component updates the reactive state service

This setup works great in many of our applications.

You can review this setup in this StackBlitz which showcases the basic principle: [Form with classic Angular @Input](https://stackblitz.com/edit/stackblitz-starters-x5kjdtku?file=src%2Fmain.ts)

## Refactor to Signal Input

With [Angular Signal Input](https://angular.dev/guide/components/inputs) we can make our component inputs reactive. This sounds great! 

FYI Signal Inputs are the recommended way for new projects. From the [Angular docs](https://angular.dev/guide/components/inputs#declaring-inputs-with-the-input-decorator): 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y8uorrbgpt7rqo9k76uf.png)

Let's refactor our classic Angular @Input to a Signal Input:
```ts
user = input.required<User>();
```

### Signal Input object value and [(ngModel)]
The Signal Input receives an object of type User. We try to bind to the object properties with `[(ngModel)]`.

This code looks so nice 😍!
```html
<div>
  <label for="firstName">First Name</label>
  <input 
    id="firstName" 
    name="firstName" 
    [(ngModel)]="user().firstName" 
  />
</div>
```
And in this StackBlitz everything seems to work: [Form with Signal Input - mutating Signal State](https://stackblitz.com/edit/stackblitz-starters-6zwfbthj?file=src%2Fmain.ts)

### ‼️ Danger Zone: 
But wait..., we have just entered the danger zone... ☢️ ☣️ ⚠️

What is actually happening?

- `user()` unwraps the Input Signal. We get hold of the raw user object
- This line of code `[(ngModel)]="user().firstName"` will **mutate** the user object (which is wrapped into a Signal) whenever the text input value changes

### ☢️ Mutating the Signal object ☣️
Why is mutating the Signal state a bad idea? Because we are bypassing the Signals public API to update state. Normally we should only use the dedicated methods `set` or `update` in order to update the Signal state.

Other developers might want to build other Signals on top of the `user` Signal with Angular `computed`. But computed will never be triggered because the `user` Signal does not know about the user object mutations. This might be a surprising behavior.

### Signal Inputs are read-only
And yes, there is another reason, why using `[(ngModel)]` on a Signal Input is at least strange. Signal Inputs are supposed to be read-only. They do not have a `set` or `update` method - so there is no official support for changing Signal Input state programmatically.

### Rescue
Let's try to escape as fast as possible 🚀. What are possible solutions?

## Linked Signal 
With [Linked Signal](https://angular.dev/guide/signals/linked-signal#) we can create a writable _user_ Signal, which is updated automatically, whenever the user Signal Input receives a new value. At the same time we can update the Linked Signal with `set` and `update`.

`editableUser` is our Linked Signal...

```ts
export class UserDetailComponent {
  user = input.required<User>();
  editableUser = linkedSignal(() => this.user()); 

  updateEditableUser(v: Partial<User>) {
    this.editableUser.update(state => ({...state, ...v}))
  } 
}
```
We also introduced a `updateEditableUser` method which uses the public Signal API to update the Signal state: in this case we call the `update` method of the Linked Signal.

```html
<div>
  <label for="firstName">First Name</label>
  <input 
    id="firstName" 
    name="firstName" 
    [ngModel]="editableUser().firstName" 
    (ngModelChange)="updateEditableUser({firstName: $event})"
  />
</div>
```
`[(ngModel)]` has been split into `[ngModel]` and `(ngModelChange)`
- `[ngModel]="editableUser().firstName"` will update the text input when the firstName property of the user object has been updated
- whenever the text input value changes, the ngModelChange callback (`updateEditableUser`) will be executed and the Linked Signal will be updated

PROs
- we use the public Signal API to update state
- we use Linked Signal which is a writable signal
- state changes happen explicitly in a dedicated method
- object clone is not needed

CONs
- There is some boilerplate necessary: Linked Signal setup, ngModel, ngModelChange, method for state update
- At the moment, Linked Signal is still in developer preview (Angular 19)

StackBlitz: [Form with Signal Input - Linked Signal](https://stackblitz.com/edit/stackblitz-starters-yhhtnn73?file=src%2Fmain.ts)

## Effect

An alternative approach is using [Angular `effect`](https://angular.dev/guide/signals#effects) to listen to new values of the Signal input.
When we receive a new value from the Input we assign the raw value to a local class property.

```ts
export class UserDetailComponent {
  _user = input.required<User>({alias: 'user'});
  user!: User;

  constructor() {
    effect(() => this.user = this._user())
  }
}
```

```html
<div>
  <label for="firstName">First Name</label>
  <input 
    id="firstName" 
    name="firstName" 
    [(ngModel)]="user.firstName" 
  />
</div>
```

In the template we can mutate the raw object as we always did. 

PROs
- The template is identical to our original old-school form component which used a classic Angular @Input
- We officially mutate a raw object (we do not bypass Signal APIs, we do not mutate the read-only Signal Input state).

CONs
- `user: User` must be initialised: `user: User = new User();` or we must tell TypeScript that user is always defined with `user!: User;`
- cloning is needed (happens in the parent with the structuredClone pipe)
- naming challenge: `_user` Signal, `'user'` alias, `user` property for the raw user object

StackBlitz: [Form with Signal Input - Effect](https://stackblitz.com/edit/stackblitz-starters-xqpmj63n?file=src%2Fmain.ts)

## @let approach

With [@let](https://angular.dev/guide/templates/variables#local-template-variables-with-let), we can declare variables in the template.

Let's use @let to get the raw object from the user Signal Input. Additionally, we can also perform the clone with our structuredClone pipe.

```html
@let userClone = user() | structuredClone;

<div>
  <label for="firstName">First Name</label>
  <input 
    id="firstName" 
    name="firstName" 
    [(ngModel)]="userClone.firstName" 
  />
</div>
```

TypeScript:
```ts
export class UserDetailComponent {
  user = input.required<User>();
}
```

PROs
- Minimum boilerplate, and smallest code change in comparison the old-school (@Input) form component
- @let is the single place to let the magic happen (unwrap the Signal, clone)

CONs
- Object value is required, it does not work with primitive values. See _Notes_ below for more details. 

StackBlitz: [Form with Signal Input - @let](https://stackblitz.com/edit/stackblitz-starters-exuqjkss?file=src%2Fmain.ts)

## Notes

### Object vs Primitive Values
In the examples above, the component Input received an object value (the user object).
The Linked Signal and Effect approaches would also work with Signal Inputs which use primitive values (string, boolean, number...).

Let's have a look at the @let approach:
[@let variables are read-only](https://angular.dev/guide/templates/variables#assignability)... which means we can not reassign new values to a @let variable. Also, not via [(ngModel)]. If we try, then we can see this compile error:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0lwigp74niuk77iuponl.png)

The @let approach requires an object value, which we can mutate.

## Conclusion
The "@let approach" seems to be the most straight-forward solution with a minimum of boilerplate.

Effect and Linked Signal are also valid options, but require more setup.

We are still evaluating which approach is most suited for our applications and this blogpost should help us to make a good decision.

I hope that you enjoyed our short visit to the danger zone of mutating Signal state. What do you think of my escape strategies? I am sure there are even more options. Let me know in the comments.

Thank you!