# Introducing MiniRx Signal Store

With the arrival of [Signals](https://angular.io/guide/signals) in Angular 16, new best practices are arising for state and events. 
In the future, Signals and Observables will coexist in your application.

Modern Angular needs modern state management which promotes **new Angular best practices** and **streamlines the usage of Signals and Observables**...

MiniRx Signal Store does exactly that.

### Modern state management with MiniRx Signal Store:

* Signal Store is an **[Angular](https://angular.dev/)-only** state management library
* Signal Store **embraces [Angular Signals](https://angular.io/guide/signals)** and leverages **Modern Angular APIs** internally
* Signal Store implements and promotes new **Angular best practices**:
    * **Signals** are used for **(synchronous) state**
    * **RxJS** is used for events and **asynchronous tasks**
* Signal Store helps to **streamline your usage of [RxJS](https://rxjs.dev/) and [Signals](https://angular.io/guide/signals)**: e.g. `connect` and `rxEffect` understand both Signals and Observables
* **Immutable Signal State**: Immutability can be enforced with the Immutable State extension
* Signal Store is based on the same great concept as the original (RxJS-based) **[MiniRx Store](https://mini-rx.io/)**
    * **All-in-one solution** for global and local state, complex and simple state
    * Three well-defined state containers:
       * Manage **global** state at large scale with the **Store (Redux) API**
       * Manage **global** state with a minimum of boilerplate using **Feature Stores**
       * Manage **local** component state with **Component Stores**
    * **Highly flexible**: Do you build complex and more simple features in the same application? You can choose the right state container individually for each feature.
    * **Simple refactor**: If you used MiniRx Store before, refactor to Signal Store will be straight-forward: change the TypeScript imports, remove the Angular async pipes (and ugly non-null assertions (`!`)) from the template
    * [**Lightweight**](https://github.com/spierala/angular-state-management-comparison) (even if you would use all three state containers together in your application)
* Signal Store has first-class support for **OOP-style** (e.g. `MyStore extends FeatureStore`), but offers also **functional creation methods** (e.g. `createFeatureStore`)

### Getting Started

#### Requirements
* Angular >= 16
* RxJS >= 7.4.0

#### Install
To install the @mini-rx/signal-store package, use your package manager of choice:

`npm install @mini-rx/signal-store`

#### API documentation
The MiniRx Signal Store API is documented in the [README](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md).

# Use-cases

MiniRx Signal Store is **highly flexible** and offers three different well-defined state containers out of the box:

- Store (Redux)
- Feature Store
- Component Store

All three can be easily used together in your application.
Depending on the use-case, you can choose the state container which suits your needs.

These are the typical use-cases:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qj5anf3ibwrjw20yu6t0.png)

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

It is true, we do not need RxJS anymore **for state**... it is time to say goodbye to `BehaviorSubject`.

But there is still an area where RxJS shines: **events** and **asynchronous tasks**.

### Why RxJS
Signals are **not suited for events**, because it is possible to miss events. See this little example using Angular `effect`:

![signal-effect-miss-event.png](assets%2Fsignal-effect-miss-event.png)

You might expect to see all state changes logged in `effect`, but that is not the case if Signal state is changed **synchronously**.

[StackBlitz](https://stackblitz.com/edit/stackblitz-starters-6qbfus?file=src%2Fmain.ts)

#### Distinct events with RxJS Subject
With a **RxJS Subject** we will be notified about **every** event (also the synchronous ones)

![event-with-subject.png](assets%2Fevent-with-subject.png)

[StackBlitz](https://stackblitz.com/edit/stackblitz-starters-fbxfuk?file=src%2Fmain.ts)

#### Side effects and race-conditions
When using RxJS-based streams, we can trigger side effects like API-calls and handle race-conditions with RxJS flattening operators (`mergeMap`, `switchMap`, `concatMap`, `exhaustMap`)

#### More operators
There is no limit! More than 100 [operators](https://rxjs.dev/guide/operators) which can be used to manipulate your streams. But to be honest, a bunch of them will bring you far: `debounceTime`, `distinctUntilChanged`, `map`, `filter`, `catchError`, etc

### RxJS in MiniRx Signal Store
Did you see the power of RxJS? Again, MiniRx Signal Store made a choice: use RxJS for events and asynchronous tasks:

- The Action stream of the (Redux) Store: implemented as RxJS Subject
- Effects: pipe the Action stream to trigger API calls (and use flattening operators to handle race-conditions)
- The `rxEffect` APIs of Feature Store and Component Store are based on RxJS Subject 


#### TODO
- Easy migration from other state management libs