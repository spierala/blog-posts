*TLDR* MiniRx is a relatively new reactive state management library. 
MiniRx "Feature Stores" offer simple yet powerful state management.
How does MiniRx Feature Store compare to @ngrx/component-store and @datorama/akita?

// TODO Quick into
// Redux, Feature Store bypss Redux...

### Quick Links
- 🤓 Learn more about MiniRx on the [Docs site](https://spierala.github.io/mini-rx-store/)
- ⭐ [MiniRx on GitHub](https://github.com/spierala/mini-rx-store)
- 🚀 See it in action on [StackBlitz](https://stackblitz.com/edit/mini-rx-store-demo)

## MiniRx vs. ComponentStore vs. Akita: FIGHT!

Let's shed some light on MiniRx by comparing it with two other popular state management libraries: @ngrx/component-store and @datorama/akita.

*Disclaimer: I am the maintainer of MiniRx Store, I try to be fair, but I it can be difficult ;)
To be clear: ComponentStore and Akita are great state management libraries. But there might be situations where MiniRx Store is just a better fit.*

## The candidates

#### Component Store (@ngrx/component-store)
ComponentStore is a library that helps to manage local/component state. It can be used as an alternative to the "Service with a Subject" approach. It is build on top of RxJS/ReplaySubject. Services which extend ComponentStore expose state as RxJS Observables (using the `select` method). With the methods `setState` and `patchState` the state can be updated.

#### Akita (@datorama/akita)
Akita describes itself as a "state management pattern": It offers a set of specialized classes like: `Store`, `Query`, `EntityStore` and more. Akita Store is build on top of RxJS/BehaviorSubject. By using the Akita classes we can build a reactive state service which exposes state as RxJS Observables (using `Query.select`). The `update` method of `Store` is used to update the state.

#### MiniRx Feature Store (mini-rx-store)
MiniRx is a "hybrid" Store. It uses Redux and RxJS BehaviorSubject under the hood and exposes the powerful Redux "Store"API (which is very similar to @ngrx/store and @ngrx/effects). At the same time MiniRx allows you to bypass the infamous Redux boilerplate with the `FeatureStore` API. You can create a reactive state service by extending FeatureStore. RxJS Observables (returned by the `select` method) inform about state changes and the state can be changed by calling `setState`.

Mhhh..., this sounds all very similar, but where are the differences then:

### 0.) Setup

Common ingredients for MiniRx Feature Store, Component Store and Akita:
```
interface CounterState {
    count: number;
}

const initialState: CounterState = {
    count: 42
}
```

All setups below will use the same state interface and initial state.

#### MiniRx Feature Store
```
@Injectable({providedIn: 'root'})
export class CounterStateService extends FeatureStore<CounterState> {

    count$: Observable<number> = this.select(state => state.count);

    constructor() {
        super('counter', initialState)
    }

    increment() {
        this.setState(state => ({count: state.count + 1}))
    }

    decrement() {
        this.setState(state => ({count: state.count - 1}))
    }
}
```
MiniRx Feature Store has to provide a "feature key" (see `super('counter', initialState)`). This is because the state is registered in the global state object.

#### Component Store
```
@Injectable({providedIn: 'root'})
export class CounterStateService extends ComponentStore<CounterState> {

    count$: Observable<number> = this.select(state => state.count);

    constructor() {
        super(initialState)
    }

    increment() {
        this.setState(state => ({count: state.count + 1}))
    }

    decrement() {
        this.setState(state => ({count: state.count - 1}))
    }
}
```

The Component Store setup looks very similar, however the "feature key" is not provided because a Component Store is local and lives independently.

#### Akita
```
@Injectable({providedIn: 'root'})
@StoreConfig({ name: 'counter' })
export class CounterStateService extends Store<CounterState> {
    constructor() {
        super(initialState)
    }

    increment() {
        this.update(state => ({count: state.count + 1}))
    }

    decrement() {
        this.update(state => ({count: state.count - 1}))
    }
}

@Injectable({providedIn: 'root'})
export class CounterQuery extends Query<CounterState> {
    count$: Observable<number> = this.select(state => state.count);

    constructor(store: CounterStateService) {
        super(store);
    }
}
```
The Akita setup is the most boilerplaty. Extending `Store` is the same as with Component Store. 
But to access the state you have to extend `Query` and provide the `Store` instance.

Also, the components need to talk to both the `Query` instance and the `Store` instance in order to read and write state.

### 1.) Local or global state.
#### MiniRx Feature Store
MiniRx at its heart is a Redux Store with one global state object ("Single source of truth"). 
MiniRx Feature Store registers a "slice" of state into the global state object.
Feature Stores are destroyable: Their state can be removed from the global state object. Therefore, Feature Stores can be used for Local Component State as well. See an example in the [Angular demo](https://angular-demo.mini-rx.io/#/counter).

#### Component Store
ComponentStore is more concerned with local state and that state is totally independent of @ngrx/store 
(which is the NgRx Redux solution and deals with global state management). 
#### Akita
The Akita Stores live independently next to each other. There is no real global state. 

### 2.) Redux Dev Tools
#### MiniRx Feature Store
Every Feature Store state becomes part of the global state object. The global state can be inspected with the Redux Dev Tools.

#### Component Store
There is no official solution for ComponentStore to inspect state with Redux Dev Tools.

#### Akita
The separate Store states are merged into one big state object to make all state inspectable with the Redux Dev Tools (https://github.com/datorama/akita/blob/master/libs/akita/src/lib/devtools.ts).

### 2.) Cross State Selectors
#### MiniRx Feature Store
Because FeatureStore state integrates into the global state object it can be selected at anytime with `store.select`.

#### Component Store
How do we combine state from different ComponentStores? By using RxJS operators like `combineLatest` or `withLatestFrom`.

#### Akita
How do we combine state from different Akita Stores? Again: By using RxJS operators like `combineLatest` or `withLatestFrom`.

### 2.) Memoized Selectors
#### MiniRx Feature Store
With MiniRx we can use memoized selectors to improve performance. At the same time the selectors are composable and reusable: https://mini-rx.io/docs/select-feature-state#memoized-selectors

This are the same selectors which are used by the MiniRx Redux Store API: https://mini-rx.io/docs/selectors#memoized-selectors

Same memoized selectors for Redux Store and FeatureStore… this also means easy refactor from using the Redux API to FeatureStore API and vice versa.

#### Component Store
Strange, no memoized selectors available? Although @ngrx/store has it.

#### Akita
No memoized selectors.

### 3.) Effects
#### MiniRx Feature Store
MiniRx FeatureStore has Effects (https://mini-rx.io/docs/effects-for-feature-store). Effects are used to trigger side effects like API calls. Again Feature Store Effects have their equivalent in the Redux API: https://mini-rx.io/docs/effects

#### Component Store
Yes, there are Effects: https://ngrx.io/guide/component-store/effect

#### Akita
There are Effects: https://datorama.github.io/akita/docs/angular/effects
However this clearly looks like an afterthought. It almost looks and feels like @ngrx/effects and does not fit so well into the rest of the Akita API.

### 4.) Undo
#### MiniRx Feature Store
MiniRx has the UndoExtension to support the undo of state changes. This is especially helpful if you want to undo optimistic updates (e.g. when an API call fails). Both the FeatureStore and the Redux API can undo specific state changes.

[small example of undoing state changes here]

#### Component Store
No support for undo.

#### Akita
Akita has a State History PlugIn to undo state changes (https://datorama.github.io/akita/docs/plugins/state-history/). The API is much bigger than the one of MiniRx. But it seems to be difficult to undo a very specific state change.

###5.) Immutable State
#### MiniRx Feature Store
MiniRx offers the ImmutableState extension to enforce immutable data. When using state management immutability is key since we only want explicit state changes using the corresponding API (e.g. by using `setState` or by dispatching an Action in the Redux API). When the ImmutableExtension is added to the MiniRx Store both the Redux API and the FeatureStore API will use immutable data.
The ImmutableExtension "deepfreezes" the global state when state is updated.

#### Component Store
There is nothing in ComponentStore which can enforce immutability.

#### Akita
Akita "deepfreezes" the state object when state is updated (only in DEV mode: https://github.com/datorama/akita/blob/master/libs/akita/src/lib/store.ts#L181

### 6.) Framework-agnostic
#### MiniRx Feature Store
MiniRx is framework-agnostic. You can use MiniRx with any framework or even without framework.

See here the MiniRx Svelte Demo: https://github.com/spierala/mini-rx-svelte-demo

#### Component Store
Component Store is tied to Angular. Angular is a peer dependency in the [package.json](https://github.com/ngrx/platform/blob/master/modules/component-store/package.json#L25).

#### Akita
Akita is also framework-agnostic. You can see in this article how Svelte and Akita play together: [Supercharge Your Svelte State Management with Akita](https://netbasal.com/supercharge-your-svelte-state-management-with-akita-f1f9de5ef43d)



// TODO major versions
