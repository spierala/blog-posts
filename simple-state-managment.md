---
description: Simple State Management in Angular with only Services and RxJS/BehaviorSubject. Let’s create our own state management Class which can be extended by Angular services.
---

*TLDR* Let’s create our own state management Class with just RxJS/BehaviorSubject (inspired by some well known state management libs).

## Manage state with RxJS BehaviorSubject
There are several great state management libraries out there to manage state in Angular: E.g. NgRx, Akita or NgXs. They all have one thing in common: They are based on RxJS Observables and the state is stored in a special kind of Observable: The BehaviorSubject.

### Why RxJS Observables?
- Observables are first class citizens in Angular. Many of the core functionalities of Angular have a RxJS implementation (e.g. HttpClient, Forms, Router and more). Managing state with Observables integrates nicely with the rest of the Angular ecosystem.
- With Observables it is easy to inform Components about state changes. Components can subscribe to Observables which hold the state. These "State" Observables emit a new value when state changes.

### What is special about BehaviorSubject?
- A BehaviorSubject emits its last emitted value to new/late subscribers
- It has an initial value
- Its current value can be accessed via the `getValue` method
- A new value can be emitted using the `next` method
- A BehaviorSubject is multicast: Internally it holds a list of all subscribers. All subscribers share the same Observable execution. When the BehaviorSubject emits a new value then the exact same value is pushed to all subscribers.

## Our own state management with BehaviorSubject
So if all the big state management libs are using RxJS BehaviorSubject and Angular comes with RxJS out of the box... Can we create our own state management with just Angular Services and BehaviorSubject?

Let's create a simple yet powerful state management Class which can be extended by Angular services.

### The key goals are:
* Be able to define a state interface and set initial state
* Straight forward API to update state and select state: `setState`, `select`
* Selected state should be returned as an Observable. The Observable emits when selected state changes.
* Be able to use `ChangeDetectionStrategy.OnPush` in our Components for better performance (read more on OnPush here: ["A Comprehensive Guide to Angular onPush Change Detection Strategy"](https://netbasal.com/a-comprehensive-guide-to-angular-onpush-change-detection-strategy-5bac493074a4)).

### The solution:
```typescript
import { BehaviorSubject, Observable } from 'rxjs';
import { distinctUntilChanged, map } from 'rxjs/operators';

export class StateService<T> {
  private state$: BehaviorSubject<T>;
  protected get state(): T {
    return this.state$.getValue();
  }

  constructor(initialState: T) {
    this.state$ = new BehaviorSubject<T>(initialState);
  }

  protected select<K>(mapFn: (state: T) => K): Observable<K> {
    return this.state$.asObservable().pipe(
      map((state: T) => mapFn(state)),
      distinctUntilChanged()
    );
  }

  protected setState(newState: Partial<T>) {
    this.state$.next({
      ...this.state,
      ...newState,
    });
  }
}
```
Let’s have a closer look at the code above:
* The StateService expects a generic type `T` representing the state interface. This type is passed when extending the StateService.
* `get state()` returns the current state snapshot
* The constructor takes an initial state and initializes the BehaviorSubject.
* `select` takes a callback function. That function is called when `state$` emits a new state. Within RxJS `map` the callback function will return a piece of state. `distinctUntilChanged` will skip emissions until the selected piece of state holds a new value/object reference.
  `this.state$.asObservable()` makes sure that the `select` method returns an Observable (and not an `AnonymousSubject`).
* `setState` accepts a Partial Type. This allows us to be lazy and pass only some properties of a bigger state interface. Inside the `state$.next` method the partial state is merged with the full state object. Finally the BehaviorSubject `this.state$` will emit a brand new state object.

### Usage
Angular Services which have to manage some state can simply extend the StateService to select and update state.

There is only one thing in the world to manage: TODOS! :) Let’s create a TodosStateService.

```typescript
interface TodoState {
  todos: Todo[];
  selectedTodoId: number;
}

const initialState: TodoState = {
  todos: [],
  selectedTodoId: undefined
};

@Injectable({
  providedIn: 'root'
})
export class TodosStateService extends StateService<TodoState>{
  todos$: Observable<Todo[]> = this.select(state => state.todos);

  selectedTodo$: Observable<Todo> = this.select((state) => {
    return state.todos.find((item) => item.id === state.selectedTodoId);
  });

  constructor() {
    super(initialState);
  }
  
  addTodo(todo: Todo) {
    this.setState({todos: [...this.state.todos, todo]})
  }

  selectTodo(todo: Todo) {
    this.setState({ selectedTodoId: todo.id });
  }
}
```
Let’s go through the TodosStateService Code:
- The TodosStateService extends `StateService` and passes the state interface `TodoState`
- The constructor needs to call `super()` and pass the initial state
- The public Observables `todos$` and `selectedTodo$` expose the corresponding state data to interested consumers like components or other services
- The public methods `addTodo` and `selectTodo` expose a public API to update state.

### Interaction with Components and Backend API
Let’s see how we can integrate our TodosStateService with Angular Components and a Backend API:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/yq8ejrf1u6j0gh3h2w23.png)

- Components call public methods of the TodosStateService to update state
- Components interested in state simply subscribe to the corresponding public Observables which are exposed by the TodosStateService.
- API calls are closely related to state. Quite often an API response will directly update the state. Therefore API calls are triggered by the TodosStateService. Once an API call has completed the state can be updated straight away using `setState`

### Demo
See a full blown TODOs App using the TodosStateService:
[Stackblitz - Angular State Manager](https://stackblitz.com/edit/rxjs-angular-state-manager?file=src/app/modules/todo/services/todos-state.service.ts)

## Notes
### Immutable Data
To benefit from `ChangeDetectionStrategy.OnPush` in our components we have to make sure to NOT mutate the state.
It is our responsibility to always pass a new object to the `setState` method. If we want to update a nested property which holds an object/array, then we have to assign a new object/array as well.

See the complete [TodosStateService (on Stackblitz)](https://stackblitz.com/edit/rxjs-angular-state-manager?file=src/app/modules/todo/services/todos-state.service.ts) for more examples of immutable state updates.

FYI
There are libs which can help you to keep the state data immutable:
[Immer](https://github.com/immerjs/immer)
[ImmutableJS](https://immutable-js.github.io/immutable-js/)

### Template Driven Forms with two-way data binding
Regarding immutable data... We have to be careful when pushing state into a Template Driven Form where the Form inputs are using `[(ngModel)]`. When the user changes a Form input value then the state object will be mutated directly...
But we wanted to stay immutable and change state only explicitly using `setState`. Therefore it is a better alternative to use Reactive Forms. If it has to be Template Driven Forms then there is still a nice compromise: one-way data binding `[ngModel]`. Another option is to (deeply) clone the form data... In that case you can still use `[(ngModel)]`.

### `async` pipe for Subscriptions
In most cases components should subscribe to the "State" Observables using the `async` pipe in the template. The async pipe subscribes for us and will handle unsubscribing automatically when the component is destroyed.

There is one more benefit of the async pipe:
When components use the OnPush Change Detection Strategy they will update their View only in these cases automatically:
- if an `@Input` receives a new value/object reference
- if a DOM event is triggered from the component or one of its children

There are situations where the component has neither a DOM event nor an @Input that changes. If that component subscribed to state changes inside the component Class, then the Angular Change Detection will not know that the View needs to be updated once the observed state emits.

You might fix it by using `ChangeDetectorRef.markForCheck()`. It tells the ChangeDetector to check for state changes anyway (in the current or next Change Detection Cycle) and update the View if necessary.

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class TodoShellComponent {
  todos: Todo[];

  constructor(
    private todosState: TodosStateService,
    private cdr: ChangeDetectorRef
  ) {
    this.todosState.todos$.subscribe(todos => {
      this.todos = todos;
      this.cdr.markForCheck(); // Fix View not updating
    });
  }
}
```
But we can also use the `async` pipe in the template instead. It is calling `ChangeDetectorRef.markForCheck` for us. See here in the Angular Source: [async_pipe](https://github.com/angular/angular/blob/10.1.x/packages/common/src/pipes/async_pipe.ts#L146)

Much shorter and prettier:
```html
<todo-list [todos]="todos$ | async"></todo-list>
```
The async pipe does a lot. Subscribe, unsubscribe, markForCheck. Let's use it where possible.

See the async pipe in action in the Demo: [todo-shell.component.html](https://stackblitz.com/edit/rxjs-angular-state-manager?file=src%2Fapp%2Fmodules%2Ftodo%2Fcomponents%2Ftodo-shell%2Ftodo-shell.component.html)

### `select` callbacks are called often
We should be aware of the fact that a callback passed to the `select` method needs to be executed on every call to `setState`.
Therefore the select callback should not contain heavy calculations.

### Multicasting is gone
If there are many subscribers to an Observable which is returned by the `select` method then we see something interesting: The Multicasting of BehaviorSubject is gone... The callback function passed to the `select` method is called multiple times when state changes. The Observable is executed per subscriber.
This is because we converted the BehaviorSubject to an Observable using `this.state$.asObservable()`. Observables do not multicast.

Luckily RxJS provides an (multicasting) operator to make an Observable multicast: `shareReplay`.

I would suggest to use the shareReplay operator only where it's needed. Let's assume there are multiple subscribers to the `todos$` Observable. In that case we could make it multicast like this:

```typescript
todos$: Observable<Todo[]> = this.select(state => state.todos).pipe(
    shareReplay({refCount: true, bufferSize: 1})
);
```

It is important to use `refCount: true` to avoid memory leaks. `bufferSize: 1` will make sure that late subscribers still get the last emitted value.

Read more about multicasting operators here: [The magic of RXJS sharing operators and their differences](https://itnext.io/the-magic-of-rxjs-sharing-operators-and-their-differences-3a03d699d255)

### Facade Pattern
There is one more nice thing. The state management service promotes the [facade pattern](https://medium.com/@thomasburlesonIA/ngrx-facades-better-state-management-82a04b9a1e39): `select` and `setState` are protected functions. Therefore they can only be called inside the `TodosStateService`. This helps to keep components lean and clean, since they will not be able to use the `setState`/`select` methods directly (e.g. on a injected TodosStateService). State implementation details stay inside the TodosStateService.
The facade pattern makes it easy to refactor the TodosStateService to another state management solution (e.g. NgRx) - if you ever want to :)

## Thanks
Special thanks for reviewing this blog post:
* [Paul Moers](https://twitter.com/paulfreelance)
* [Michael Rutzer - diePartments](https://twitter.com/diePartments)
* [Jan-Niklas Wortmann - RxJS Core Team Member](https://twitter.com/niklas_wortmann)

Articles which inspired me:
* [Simple state management in Angular with only Services and RxJS](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8) by [Aslan Vatsaev](https://twitter.com/avatsaev)
* Very similar approach: [Creating A Simple setState() Store Using An RxJS BehaviorSubject In Angular 6.1.10](https://www.bennadel.com/blog/3522-creating-a-simple-setstate-store-using-an-rxjs-behaviorsubject-in-angular-6-1-10.htm) by [Ben Nadel](https://twitter.com/BenNadel)
