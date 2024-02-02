# Introducing MiniRx Signal Store

With the arrival of [Signals](https://angular.io/guide/signals) in Angular 16, new best practices are arising for state and events. 
In the future, Signals and Observables will coexist in your application.

Modern Angular needs modern state management which promotes **new Angular best practices** and **streamlines the usage of Signals and Observables**...

MiniRx Signal Store does exactly that.

### Modern state management with MiniRx Signal Store:

* Signal Store **embraces [Angular Signals](https://angular.io/guide/signals)** and leverages **Modern Angular APIs** internally
* Signal Store implements and promotes new **Angular best practices**:
    * **Signals** are used for **(synchronous) state**
    * **RxJS** is used for events and **asynchronous tasks**
* Signal Store **streamlines your usage of [RxJS](https://rxjs.dev/) and [Signals](https://angular.io/guide/signals)**: e.g. `connect` and `rxEffect` understand both Signals and Observables
* Signal Store is based on the same great concept as the original (RxJS-based) **[MiniRx Store](https://mini-rx.io/)**
    * It is an **all-in-one solution** for global and local state, complex and simple state
    * You get three well-defined state containers: **Store (Redux), Feature Store** and  **Component Store**
    * **Highly flexible**: Do you build complex and more simple features in the same application? You can choose the right state container individually for each feature.

### Getting Started

#### Requirements
* Angular >= 16
* RxJS >= 7.4.0

#### Install
To install the @mini-rx/signal-store package, use your package manager of choice:

`npm install @mini-rx/signal-store`

#### API documentation
The MiniRx Signal Store API is documented in the [README](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md).

# Evolution
...instead of revolution! MiniRx tries to make the transition to signal-based state management as smooth as possible:
- There is no need to learn new concepts (most MiniRx Signal Store concepts are known from traditional RxJS-based state management libraries)
- MiniRx Signal Store APIs are very similar to the original (RxJS-based) MiniRx Store

### OOP-style
MiniRx Signal Store supports the good old **Object-oriented programming** style which many people appreciate in Angular.

`class MyStore extends FeatureStore<ProductState>{}`

### Functional style
You can use functional creation methods when extending a class would feel like overkill:
e.g. if you want to create a little store just inline in your component code.

`createFeatureStore`, `createComponentStore`

### Classic Module APIs and modern Standalone APIs
MiniRx Signal Store supports both classic Angular Module APIs (e.g. `StoreModule.forRoot()`) and modern Standalone APIs (e.g. `provideStore`).

### Easy refactor
If you used the original MiniRx Store before, refactor to MiniRx Signal Store will be pretty straight-forward: 
- change the TypeScript imports (import from `@mini-rx/signal-store` instead of `mini-rx-store`)
- `select` methods return Signal instead of Observable 

In fact, it is possible to refactor to MiniRx Signal Store from traditional state management libraries like Akita or NgRx Store. A lot of MiniRx Signal Store APIs are very similar.

# All-in-one solution
MiniRx Signal Store is an all-in-one solution and offers three different well-defined state containers out of the box:

* Manage **global** state at large scale with the **[Store (Redux) API](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md#redux-api)**
* Manage **global** state with a minimum of boilerplate using **[Feature Stores](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md#feature-store-api)**
* Manage **local** component state with **[Component Stores](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md#component-store-api)**

## Flexibility
All three state containers can be easily used together in your application.
Depending on the use-case, you can choose the state container which suits your needs.

These are the typical use-cases:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qj5anf3ibwrjw20yu6t0.png)

## High level of integration
MiniRx Signal Store was designed as an all-in-one solution from the beginning. This allows for a high level of integration:
- Feature Store uses the Redux Store under the hood
- Feature Store state integrates into the global state object of the Redux Store
- Feature Store automatically uses all the extensions of the Redux Store (e.g. Redux DevTools extension, ImmutableState extension)
- Feature Store and Component Store share the same API
- Learning one state container helps learning the rest of the bunch:
  - Do you like the memoized selectors of the Redux Store? Feel free to use them with Feature Store and Component Store!
  - Do you trigger API calls in effects using the Redux Store? You can be sure there will be something for effects in Feature Store and Component Store: `rxEffect`
  - Did you master the Feature Store API? You mastered Component Store at the same time! Both share the same API.

## Lightweight
Because MiniRx Signal Store is an all-in-one solution, it can be a very **lightweight library**.

A lot of code is shared internally:
- Feature Store uses the existing Redux Store under the hood: If you use Feature Store already, and you start using the Redux Store later, the app bundle size will not increase significantly.
- Feature Store and ComponentStore share the same methods (e.g. `connect` and `rxEffect`)

_FYI_ MiniRx Signal Store if one of the most lightweight state management libraries in the [Angular state management bundle size comparison](https://github.com/spierala/angular-state-management-comparison).

### Tree-shakeable
MiniRx Signal Store is designed to be tree-shakeable as much as possible. If you use only the Component Store... nothing from the Redux Store will be part of the bundle.

# New Angular best-practices in MiniRx Signal Store

The MiniRx Signal Store is implemented using new Angular best-practices.
At the same time MiniRx Signal Store also promotes these best-practices in its new API:
* **Signals** are used for **(synchronous) state**
* **RxJS** is used for events and **asynchronous tasks**

Let's read on to understand why MiniRx Signal Store made this choice...

## Signals
In version 16, Angular introduced **Signals** as a new **reactive primitive**. Prior to this, **RxJS** was the go-to tool for managing state in a reactive manner.

### Why Signals?
Signals in Angular have some advantages, compared to RxJS:

* Write subscription-free code, even without using the `async` pipe
* Easier to learn (no pipe, no operators, Signals are always synchronous)
* Easier to compose derived state from other Signals with `computed` instead of RxJS `combineLatest`
* Signals may enable more efficient Angular Change Detection in the future

### Signals in MiniRx Signal Store
Hopefully, you agree that Signals are the new best choice for state!

MiniRx Signal Store definitely made the choice: it uses Angular Signal internally and exposes Signals via its public API:
- The global state object of the Redux Store (which is also used by the Feature Store) is implemented as Angular Signal
- public API: all three state containers have a `select` method: it returns an Angular Signal
- Memoized selectors (used to select state from the global state object) are implemented using Angular `computed`

## RxJS
You may ask: Why do we still need RxJS? We have Signals now!

It is true, we do not need RxJS anymore **for state**: it is time to say goodbye to `BehaviorSubject`!

But there is still an area where RxJS shines: **events** and **asynchronous tasks**.

### Distinct events with RxJS Subject
Signals are **not suited for events**, because it is possible to miss events. See this little example using Angular `effect`:

![signal-effect-miss-event.png](assets%2Fsignal-effect-miss-event.png)

[StackBlitz](https://stackblitz.com/edit/stackblitz-starters-6qbfus?file=src%2Fmain.ts)

You might expect to see all state changes logged in `effect`, but that is not the case if Signal state is changed **synchronously**!

With a **RxJS Subject** we will be notified about **every** event (also the synchronous ones).

![event-with-subject.png](assets%2Fevent-with-subject.png)

[StackBlitz](https://stackblitz.com/edit/stackblitz-starters-fbxfuk?file=src%2Fmain.ts)

### Side effects and race-conditions
When using RxJS-based streams, we can trigger side effects like API-calls and handle race-conditions with RxJS flattening operators (`mergeMap`, `switchMap`, `concatMap`, `exhaustMap`).

### More operators
There is no limit! More than 100 [operators](https://rxjs.dev/guide/operators) which can be used to manipulate your streams.
But to be honest, even a small bunch of operators will take you far: `debounceTime`, `distinctUntilChanged`, `map`, `filter`, `catchError`, etc

### RxJS in MiniRx Signal Store
Did you see the strengths of RxJS? 

MiniRx Signal Store made its choice... use RxJS for events and asynchronous tasks:
- The Action stream of the (Redux) Store: implemented as RxJS Subject
- Effects: pipe the Action stream to trigger API calls (and use flattening operators to handle race-conditions)
- The `rxEffect` APIs of Feature Store and Component Store are based on RxJS Subject 

# RxJS and Signal Interop

MiniRx Signal Store will help you to streamline the usage of RxJS Observables and Signals.
The goal is to eliminate any conversion code in your application: say goodbye to `toSignal` and to `toObservable`!

These MiniRx Signal Store APIs can handle both Observables and Signals:

### `rxEffect`

`rxEffect` is used to trigger side effects like API calls in Feature Store and Component Store.
There are three different ways to trigger the side effect:

- Raw Value
- Signal
- Observable

Following (Component Store) example uses an Angular Signal (Input) to trigger the API call:

```ts
import { Component, inject, input, Signal } from '@angular/core';
import { createComponentStore, tapResponse } from '@mini-rx/signal-store';
import { switchMap } from 'rxjs';
import { BookService } from '../book.service';

type State = {
  detail: BookDetail;
  isLoading: boolean;
}

const initialState: State = {
  detail: undefined,
  isLoading: false
}

@Component({
// ...
})
export class BookComponent {
  private store = createComponentStore(initialState);
  private bookService = inject(BookService);

  bookId = input.required<string>(); // Signal Input

  bookDetail: Signal<BookDetail> = this.store.select(state => state.detail);
  isLoading: Signal<boolean> = this.store.select(state => state.isLoading);

  // Create an Effect
  private loadDetail = this.store.rxEffect<string>(
    // Handle race-condition with switchMap
    switchMap(id => {
      this.store.setState({isLoading: true});

      return this.bookService.getBookDetail(id).pipe(
        tapResponse({
          next: (detail: BookDetail) => {this.store.setState({detail})},
          error: () => this.store.setState({isLoading: false})
        })
      )
    })
  )

  constructor() {
    // Fetch detail for every new bookId Signal value
    this.loadDetail(this.bookId)
  }
}
```

### `connect`
Available in Feature Store, Component Store.

With `connect` you have the possibility to connect your store with external sources like Observables and Signals.
This helps to make your store the Single Source of Truth for your state.

We are connecting the Component Store to both Observable and Signal in this example:
```ts
import { Component, Signal, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import { createComponentStore } from '@mini-rx/signal-store';
import { timer } from 'rxjs';

@Component({
// ...
})
export class ConnectComponent {
  store = createComponentStore({
    counter: 0,
    counterFromObservable: 0, // Will be updated via Observable
    counterFromSignal: 0, // Will be updated via Signal
  });

  sum: Signal<number> = this.store.select((state) => {
    return state.counter + state.counterFromObservable + state.counterFromSignal;
  });

  constructor() {
    const interval = 1000;

    const observableCounter$ = timer(0, interval); // Observable
    const signalCounter = signal(0); // Signal

    // Connect external sources (Observables or Signals) to the Component Store
    this.store.connect({
      counterFromObservable: observableCounter$, // Observable
      counterFromSignal: signalCounter, // Signal
    });

    setInterval(() => signalCounter.update((v) => v + 1), interval);
  }

  increment() {
    this.store.setState((state) => ({ counter: state.counter + 1 }));
  }
}
```

## Destroyable Stores
By using Angular `DestroyRef` internally, Feature Stores and Component Stores know their creation context... they will get notified when the creation context is destroyed. 
Now you can safely create Feature Stores and Component Stores inside your component code.
When the component is destroyed, the stores are also destroyed. Also, any internal subscriptions (e.g. from `rxEffect` or `connect`) will be unsubscribed.

## Framework-agnostic code
If you look closer at the MiniRx Signal Store, you will see an interesting aspect:
The Angular Signal APIs (`signal`, `computed`, `effect`, `toSignal`, `toObservable`) become (almost) an implementation detail of the MiniRx Signal Store.
Only the `select` methods return Signal. That's it. 

In fact, with MiniRx Signal Store you are writing largely framework-agnostic code! MiniRx Signal Store code can be easily refactored to the framework-agnostic original MiniRx Store.

## Summary
We have seen, MiniRx Signal Store is an incredibly flexible state management solution: 
It does not matter if you manage global or local state, complex or simple state... MiniRx Signal Store has you covered!

These are exciting times for Angular: old best-practices disappear, new ones appear...
The opinionated nature of MiniRx Signal Store will help you find the right way through modern Angular.

## ⭐ MiniRx on GitHub
Do you like MiniRx? Give it a GitHub star [here](https://github.com/spierala/mini-rx-store).

Thank you! :)

## Demos
MiniRx Signal Store was successfully tested in these projects:

- [Angular Tetris](https://github.com/trungvose/angular-tetris/pull/45)
- [Angular Jira Clone](https://github.com/trungvose/jira-clone-angular/pull/99)
- [MiniRx Signal Store Demo](https://signal-store-demo.mini-rx.io/)

## Release
MiniRx Signal Store 1.0.0 was published today!

# TODO
- Thanks to blogpost reviewers / code reviewers
- Mention immutable signal state?
- Show Redux DevTools?
- Revisit Summary
- Release Signal Store v1 before publishing the blogpost! 