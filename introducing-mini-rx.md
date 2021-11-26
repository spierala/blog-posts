## Hello from MiniRx Store
**MiniRx Store** is the new kid on the reactive state management block. MiniRx will help you to manage state at large scale (with **Redux**), but it also offers a simple form of state management: **Feature Stores**.

Let's get a quick overview:

### Global
MiniRx Store is a **global**, application-wide solution to manage state in **JavaScript** and **TypeScript** applications.

### Reactive
MiniRx is powered by **[RxJS](https://github.com/ReactiveX/rxjs)** and exposes **state as RxJS Observables**. The Observables emit when their selected state changes.

### Scalable
MiniRx is a full-blown **Redux** Store: It includes actions, reducers, meta reducers, memoized selectors and redux dev tools support.
However, with MiniRx **Feature Stores** we can **bypass Redux boilerplate**: update state straight away with `setState`.

MiniRx scales nicely with your state management requirements:
- Make hard things simple with the **Redux** API
- Keep simple things simple with the **Feature Store** API

### Quick Links
- 🤓 Learn more about MiniRx on the [Docs site](https://spierala.github.io/mini-rx-store/)
- ⭐ [MiniRx on GitHub](https://github.com/spierala/mini-rx-store)
- 🚀 See it in action on [StackBlitz](https://stackblitz.com/edit/mini-rx-store-demo)

## The MiniRx DNA
### NgRx
MiniRx is inspired by [NgRx](https://ngrx.io/), which is a well known reactive Redux Store in the [Angular](https://angular.io/) world. MiniRx and NgRx look similar at first sight, but there are also important differences.

### What do NgRx and MiniRx have in common?
* Their names sound similar: try to speak them out loudly ;)
* Powered by **RxJS**
* Both implement the **Redux Pattern**
* State and actions are exposed as **RxJS Observable**

### What are the main differences?
* MiniRx has no Angular dependency and is **framework agnostic**
* MiniRx has **Feature Stores** to manage feature state **without Redux boilerplate**
* MiniRx is more **lightweight** and admittedly does not cover every crazy use case of state management

And it is these differences which explain the name "MiniRx".

## Why MiniRx
NgRx and the Redux pattern are well-suited for managing state at a large scale. But almost every application contains also features which require only a **simple form of state management**. Then the Redux pattern with its actions and reducers quickly feels like **overkill**. It would be great to have a state management solution which looks and feels a lot like NgRx, but it has to support simple state management too: **Scalable state management**. It was time to create [MiniRx Store](https://github.com/spierala/mini-rx-store):
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pjybr0c4zsanlimdaltn.png)

### Bypass Redux boilerplate with Feature Stores
MiniRx uses the Redux pattern to make state management explicit and predictable. The Redux pattern is very powerful, but it comes with some boilerplate code (mostly actions, reducers, dispatching actions). MiniRx allows us to bypass the Redux boilerplate for simple feature states: With **Feature Stores** we can manage feature state directly **without actions and reducer** (simply use `setState` to update the feature state).

### State Management which scales
For simple features we can use Feature Stores. And Feature Stores can be quite powerful actually: You can use **memoized selectors** (if you want to), you can create **effects** for API calls (if you want to). MiniRx scales nicely with your needs.
And you can always fall back to the Redux API in case that you have to manage huge and complex state.

### Framework agnostic
NgRx is a great reactive Store, but it currently only works in Angular. There are also other frontend frameworks like [Svelte](https://svelte.dev/) which embrace reactivity. It would be cool to do NgRx-style state management in Svelte!
With MiniRx you can use whatever framework you want: you can build a **framework-agnostic** state management layer for Angular today and move it to Svelte (or any other frontend framework) tomorrow.

## MiniRx Key Concepts
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ssx0l49baspywefsp141.PNG)

* The store is a single object which holds the global application state. It is the **"single source of truth"**
* State has a **flat hierarchy** and is divided into **"feature states"** (also called "slices" in Redux world)
* For each "feature state" we can decide to use the **Redux API** with actions and a reducer or the **Feature Store API** with `setState`
* State is exposed as **RxJS Observable** (The Store exposes the **global state**, while a Feature Store exposes a specific **feature state**).
* State is **read-only** (immutable) and can only be changed by dispatching actions (Redux API) or by using `setState` (Feature Store API)

That was a long introduction! Let's dive into some code to see MiniRx in action...

## Basic Tutorial

### Store (Redux API)
MiniRx supports the classic Redux API with registering **reducers** and dispatching **actions**.
**Observable state** can be selected with **memoized selectors**.

```ts
import {
  Action,
  Store,
  configureStore,
  createFeatureSelector,
  createSelector
} from 'mini-rx-store';
import { Observable } from 'rxjs';

// 1.) State interface
interface CounterState {
  count: number;
}

// 2.) Initial state
const counterInitialState: CounterState = {
  count: 1
};

// 3.) Reducer
function counterReducer(
  state: CounterState = counterInitialState,
  action: Action
): CounterState {
  switch (action.type) {
    case 'inc':
      return {
        ...state,
        count: state.count + 1
      };
    default:
      return state;
  }
}

// 4.) Get hold of the store instance and register root reducers
const store: Store = configureStore({
  reducers: {
    counter: counterReducer
  }
});

// 5.) Create memoized selectors
const getCounterFeatureState = createFeatureSelector<CounterState>('counter');
const getCount = createSelector(
  getCounterFeatureState,
  state => state.count
);

// 6.) Select state as RxJS Observable
const count$: Observable<number> = store.select(getCount);
count$.subscribe(count => console.log('count:', count));

// 7.) Dispatch an action
store.dispatch({ type: 'inc' });

// OUTPUT: count: 1
// OUTPUT: count: 2
```
### Feature Store API
Feature Stores allow us to manage feature state **without actions and reducers**.
The API of a Feature Store is optimised to select and update a feature state directly with a **minimum of boilerplate**.

```ts
import { FeatureStore } from 'mini-rx-store';
import { Observable } from 'rxjs';

// 1.) State interface
interface CounterState {
  count: number;
}

// 2.) Initial state
const counterInitialState: CounterState = {
  count: 11
};

export class CounterFeatureStore extends FeatureStore<CounterState> {
  // Select state as RxJS Observable
  count$: Observable<number> = this.select(state => state.count);

  constructor() {
    super('counterFs', counterInitialState);
  }

  // Update state with `setState`
  inc() {
    this.setState(state => ({ ...state, count: state.count + 1 }));
  }
}
```

Use the "counterFs" feature store like this:
```ts
import { CounterFeatureStore } from "./counter-feature-store";

const counterFs = new CounterFeatureStore();
counterFs.count$.subscribe(count => console.log('count:', count));
counterFs.inc();

// OUTPUT: count: 11
// OUTPUT: count: 12
```

### The state of a Feature Store becomes part of the global state

Every new Feature Store will show up in the global state with the corresponding feature key (e.g. "counterFs"):

```ts
store.select(state => state).subscribe(console.log);

//OUTPUT: {"counter":{"count":2},"counterFs":{"count":12}}
```
## Notes
### Demo
🚀 See MiniRx Store in action on [StackBlitz](https://stackblitz.com/edit/mini-rx-store-demo)

The Demo uses both the Redux API and the Feature Stores:
- Todos: Feature Store
- Products and Cart: Redux
- User: Feature Store

### More MiniRx Examples:
These popular Angular demo applications show the power of MiniRx:
- [Angular Tetris with MiniRx](https://github.com/spierala/angular-tetris-mini-rx)
- [Angular Jira Clone using MiniRx](https://github.com/spierala/jira-clone-angular)
- Coming soon: Angular Spotify using MiniRx

### Documentation
Check out the [docs](https://spierala.github.io/mini-rx-store/) for the full MiniRx API.

### Show Your Support
If you like MiniRx: Give it a ⭐ on [GitHub](https://github.com/spierala/mini-rx-store)

### References
These projects, articles and courses helped and inspired me to create MiniRx:

-   [NgRx](https://ngrx.io/)
-   [Akita](https://github.com/datorama/akita)
-   [Observable Store](https://github.com/DanWahlin/Observable-Store)
-   [RxJS Observable Store](https://github.com/jurebajt/rxjs-observable-store)
-   [Juliette Store](https://github.com/markostanimirovic/juliette)
-   [Basic State Management with an Observable Service](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8)
-   [Redux From Scratch With Angular and RxJS](https://www.youtube.com/watch?v=hG7v7quMMwM)
-   [How I wrote NgRx Store in 63 lines of code](https://medium.com/angular-in-depth/how-i-wrote-ngrx-store-in-63-lines-of-code-dfe925fe979b)
-   [NGRX VS. NGXS VS. AKITA VS. RXJS: FIGHT!](https://ordina-jworks.github.io/angular/2018/10/08/angular-state-management-comparison.html?utm_source=dormosheio&utm_campaign=dormosheio)
-   [Pluralsight: Angular NgRx: Getting Started](https://app.pluralsight.com/library/courses/angular-ngrx-getting-started/table-of-contents)
-   [Pluralsight: RxJS in Angular: Reactive Development](https://app.pluralsight.com/library/courses/rxjs-angular-reactive-development/table-of-contents)
-   [Pluralsight: RxJS: Getting Started](https://app.pluralsight.com/library/courses/rxjs-getting-started/table-of-contents)


