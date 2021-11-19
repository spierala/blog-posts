*TLDR* MiniRx is a relativly new reactive state management library. How does it compare to @ngrx/component-store and @datorama/akita?

## MiniRx in short:
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

## Why MiniRx Store?
Let's compare MiniRx with its main competitors to get clarity: @ngrx/component-store and @datorama/akita.

*Disclaimer: I am the maintainer of MiniRx Store, I try to be fair, but I it can be difficult ;)
To be clear: ComponentStore and Akita are great state management libraries. But there might be situations where MiniRx Store is just a better fit.*

## The candidates

#### @ngrx/component-store
ComponentStore is a library that helps to manage local/component state. It can be used as an alternative to the "Service with a Subject" approach. It is build on top of RxJS/ReplaySubject. Services which extend ComponentStore expose state as RxJS Observables (using the `select` method). With the methods `setState` and `patchState` the state can be updated.

#### @datorama/akita
Akita describes itself as a "state management pattern": It offers a set of specialized classes like: `Store`, `Query`, `EntityStore` and more. Akita Store is build on top of RxJS/BehaviorSubject. By using the Akita classes we can build a reactive state service which exposes state as RxJS Observables (using `Query.select`). The `update` method of `Store` is used to update the state.

#### mini-rx-store
MiniRx is a "hybrid" Store. It uses Redux and RxJS BehaviorSubject under the hood and exposes the powerful Redux "Store"API (which is very similar to @ngrx/store and @ngrx/effects). At the same time MiniRx allows you to bypass the infamous Redux boilerplate with the `FeatureStore` API. You can create a reactive state service by extending FeatureStore. RxJS Observables (returned by the `select` method) inform about state changes and the state can be changed by calling `setState`.

Mhhh this sounds all very similar, but there are significant differences:

## MiniRx vs. ComponentStore vs. Akita: FIGHT!

### 1.) Hybrid Store and global state
#### mini-rx-store
MiniRx as a "hybrid" Store exposes the Redux and the FeatureStore API:

MiniRx Store (Redux API):
MiniRx supports the classic Redux API with registering reducers and dispatching actions.

MiniRx Feature Store API:
FeatureStores allow us to manage feature state without actions and reducers.
The API of a Feature Store is optimised to select and update a feature state directly with a minimum of boilerplate.

We can use `FeatureStore` for the more simple features of the application while we can use Redux for the really big and complex features.

`StoreModule.forFeature('products', productsReducer)` and `new FeatureStore('todos', initialState)` both add a new "Slice" of state to the global state object.

Because FeatureStore state integrates into the global state object it can be selected at anytime with `store.select`. Global state can be inspected with the Redux Dev Tools.

Combining the different "Slices" of state is a question of writing a global selector function which can be passed to `store.select`.

[Image of Redux Dev Tools here]

####@ngrx/component-store
ComponentStore is really only concerned with local state and that state is totally independent from @ngrx/store (Redux, global state management). Therefore it is not possible to inspect ComponentStore state with Redux Dev Tools.

How do we combine state from different ComponentStores? By using RxJS operators like `combineLatest` or `withLatestFrom`.

####@datorama/akita
The Akita Stores live independently next to each other. There is no real global state. However the separate Store states are merged into one big state object to make all state inspectable with the Redux Dev Tools (https://github.com/datorama/akita/blob/master/libs/akita/src/lib/devtools.ts).

How do we combine state from different Akita Stores? Again: By using RxJS operators like `combineLatest` or `withLatestFrom`.

### 2.) Memoized Selectors
#### mini-rx-store
With MiniRx we can use memoized selectors to improve performance. At the same time the selectors are composable and reusable: https://mini-rx.io/docs/select-feature-state#memoized-selectors

This are the same selectors which are used by the MiniRx Redux Store API: https://mini-rx.io/docs/selectors#memoized-selectors

Same memoized selectors for Redux Store and FeatureStore… this also means easy refactor from using the Redux API to FeatureStore API and vice versa.

####@ngrx/component-store
Strange, no memoized selectors available? Although @ngrx/store has it.

####@datorama/akita
No memoized selectors.

### 3.) Effects
####mini-rx-store
MiniRx FeatureStore has Effects (https://mini-rx.io/docs/effects-for-feature-store). Effects are used to trigger side effects like API calls. Again Feature Store Effects have their equivalent in the Redux API: https://mini-rx.io/docs/effects

####@ngrx/component-store
Yes, there are Effects: https://ngrx.io/guide/component-store/effect

####@datorama/akita
There are Effects: https://datorama.github.io/akita/docs/angular/effects
However this clearly looks like an afterthought. It almost looks and feels like @ngrx/effects and does not fit so well into the rest of the Akita API.

###4.) Undo
####mini-rx-store
MiniRx has the UndoExtension to support the undo of state changes. This is especially helpful if you want to undo optimistic updates (e.g. when an API call fails). Both the FeatureStore and the Redux API can undo specific state changes.

[small example of undoing state changes here]

####@ngrx/component-store
No support for undo.

####@datorama/akita
Akita has a State History PlugIn to undo state changes (https://datorama.github.io/akita/docs/plugins/state-history/). The API is much bigger than the one of MiniRx. But it seems to be difficult to undo a very specific state change.

###5.) Immutable State
####mini-rx-store
MiniRx offers the ImmutableState extension to enforce immutable data. When using state management immutability is key since we only want explicit state changes using the corresponding API (e.g. by using `setState` or by dispatching an Action in the Redux API). When the ImmutableExtension is added to the MiniRx Store both the Redux API and the FeatureStore API will use immutable data.
The ImmutableExtension "deepfreezes" the global state when state is updated.

####@ngrx/component-store
There is nothing in ComponentStore which can enforce immutability.

####@datorama/akita
Akita "deepfreezes" the state object when state is updated (only in DEV mode: https://github.com/datorama/akita/blob/master/libs/akita/src/lib/store.ts#L181
