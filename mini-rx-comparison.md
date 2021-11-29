---
description: MiniRx Feature Store offers simple yet powerful state management in Angular. How does MiniRx Feature Store compare to NgRx Component Store and Akita?
---

**MiniRx "Feature Stores"** offer **simple yet powerful state management**.
How does **MiniRx Feature Store** compare to **@ngrx/component-store** and **@datorama/akita**? 10 rounds in the **fighting ring** will bring clarity!

*Disclaimer: I am the maintainer of MiniRx Store, I try to be fair, but it can be difficult from time to time.
To be clear: Component Store and Akita are great state management libraries. It will be an intense fight, but I will make sure that nobody gets hurt!*

## What is MiniRx?
MiniRx is a full-blown **Redux** Store powered by **RxJS**: It includes actions, reducers, meta reducers, memoized selectors, effects and Redux DevTools support.

The Redux pattern is great to manage state at large scale, but it forces us to write boilerplate code (actions, reducers, dispatch actions). This can be overkill for simple features in your application.

For that reason, MiniRx **Feature Store** offers a more simple form of state management: we can **bypass Redux boilerplate** and interact **directly** with a corresponding **feature state** with the `FeatureStore` API:
- `setState()` update the feature state
- `select()` select state from the feature state object as RxJS Observable
- `effect()` run side effects like API calls and update feature state
- `undo()` easily undo setState actions (requires the UndoExtension)
- `get state()` imperatively get the current feature state

MiniRx scales nicely with your state management requirements:

- Make hard things simple with the Redux `Store` API
- Keep simple things simple with the `FeatureStore` API

In most cases you can default to the `FeatureStore` API and fall back to the Redux `Store` API to implement the really complex features in your application.

#### How does the Feature Store work?
Feature Store uses Redux under the hood:
Behind the scenes a Feature Store is creating a _feature reducer_ and a corresponding _setState_ action.
The _feature reducer_ is registered in the Redux Store and the Feature Store state becomes part of the global state object.
When calling `setState()` the Feature Store dispatches its _setState_ action (with the new state as action payload) and the _feature reducer_ will update the feature state accordingly.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2wgzrqjoz5ikedd3by3h.png)

See the `FeatureStore` source [here](https://github.com/spierala/mini-rx-store/blob/3.0.0/projects/mini-rx-store/src/lib/feature-store.ts).

#### Links
- 🤓 Learn more about MiniRx on the [docs site](https://mini-rx.io)
- ⭐ [MiniRx on GitHub](https://github.com/spierala/mini-rx-store)
- 🚀 See it in action in the [Angular Demo](https://angular-demo.mini-rx.io/)
- 🤓 [Feature Store docs](https://mini-rx.io/docs/fs-quick-start)
- 🚀 [MiniRx Basic Tutorial on StackBlitz](https://stackblitz.com/edit/mini-rx-store-basic-tutorial?file=index.ts):
  See how the Redux API and Feature Store API both add to the global state object

## MiniRx Feature Store vs. NgRx Component Store vs. Akita

Let's shed some light on **MiniRx Feature Store** by sending it to the fighting ring together with two other popular state management libraries: **@ngrx/component-store** and **@datorama/akita**.

## The competitors

#### NgRx Component Store (13.0.1)
Component Store is a library that helps to manage local/component state. It can be used as an alternative to the "Service with a Subject" approach. 
It is build on top of RxJS/ReplaySubject (see [here](https://github.com/ngrx/platform/blob/13.0.1/modules/component-store/src/component-store.ts#L52)). Services which extend `ComponentStore` expose state as RxJS Observables (using the `select` method). With the methods `setState` and `patchState` the state can be updated.

Docs: https://ngrx.io/guide/component-store

#### Akita (6.2.4)
Akita describes itself as a "state management pattern": 
It offers a set of specialized classes like `Store`, `Query`, `EntityStore` and more. 
Akita `Store` is build on top of RxJS/BehaviorSubject (see [here](https://github.com/datorama/akita/blob/v6.2.0/libs/akita/src/lib/store.ts#L49)). 
By using the Akita classes we can build a reactive state service which exposes state as RxJS Observables (using `select` on a `Query` instance). The `update` method of `Store` is used to update the state.

Docs: https://datorama.github.io/akita/

#### MiniRx Feature Store (3.0.0)
MiniRx itself is a "hybrid" Store. It uses Redux and RxJS/BehaviorSubject (see [here](https://github.com/spierala/mini-rx-store/blob/3.0.0/projects/mini-rx-store/src/lib/store-core.ts#L30)) under the hood and exposes the powerful Redux `Store` API (which is very similar to [@ngrx/store](https://ngrx.io/guide/store) and [@ngrx/effects](https://ngrx.io/guide/effects)). 
At the same time MiniRx allows you to bypass the infamous Redux boilerplate with the `FeatureStore` API. 
You can create a reactive state service by extending `FeatureStore`. 
RxJS Observables (returned by the `select` method) inform about state changes and the state can be changed by calling `setState`.

Docs: https://mini-rx.io/docs/fs-quick-start

Mhhh..., this sounds all very similar, but where are the differences then? It's time to prepare the fighting ring! :)

## FIGHT!
10 rounds to go! 

### 1. Basic Setup

What does the basic setup of a **reactive state service** look like?

All setups share the same ingredients: A state interface and initial state. 

FYI: The state interface must be object-like: you can not state-manage just a plain `number` or `string`. 
```typescript
interface CounterState {
    count: number;
}

const initialState: CounterState = {
    count: 42
}
```

#### MiniRx Feature Store
The state service extends `FeatureStore`:
```typescript
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
MiniRx Feature Store has to provide the initial state and a _feature key_: "counter". 
The key is used to register the "counter" state in the global state object.

#### Component Store
With Component Store we extend `ComponentStore` and provide an initial state:
```typescript
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
The Component Store setup looks very similar to Feature Store, however the feature key is not needed because every `ComponentStore` instance lives independently.
FYI: The Component Store initial state parameter is optional (see docs [here](https://ngrx.io/guide/component-store/initialization#lazy-initialization)).

#### Akita
With Akita, we create two services: One extends `Store` and the other one extends `Query`:
```typescript
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
The Akita setup is the most boilerplaty. Extending `Store` is similar to the other setups. A feature key is provided via the `@StoreConfig` decorator.
To access the state you have to extend `Query` and provide the `Store` instance.
Also, the components have to talk to both the `Query` and the `Store` instance in order to read and write state.

### 2. Bundle Sizes
Regarding the basic setup..., let's look at the corresponding bundle sizes (using [source-map-explorer](https://www.npmjs.com/package/source-map-explorer)). 

#### MiniRx Feature Store
combined: 152.39 KB

#### Component Store
combined: 152.25 KB

#### Akita
combined: 151.61 KB

Akita is the most lightweight, and MiniRx is almost 1 KB bigger. 
But keep in mind that MiniRx Feature Store uses Redux under the hood 
and the Redux API is always available. Using the MiniRx Redux API will not add much to the total bundle size.

### 2.1. Bundle Sizes when adding Redux

#### MiniRx Feature Store + Store API (Store + Effects) using [Angular Integration (mini-rx-store-ng)](https://mini-rx.io/docs/angular#register-effects)
combined: 156.9 KB

#### NgRx Component Store + NgRx Store
combined: 164.17 KB

#### NgRx Component Store + NgRx Store + NgRx Effects
combined: 171.45 KB

You can review the different setups in this repo and run source-map-explorer yourself: https://github.com/spierala/mini-rx-comparison

### 3. Local or global state
How do the different store solutions relate to local (component state) and global state? What is the store lifespan?

#### MiniRx Feature Store
MiniRx at its heart is a Redux Store with one global state object ("Single source of truth"). Also, MiniRx Feature Stores register a "slice" of state into the global state object.
The focus of MiniRx is clearly global state which has the lifespan of the application.

But Feature Stores are destroyable... Their state can be removed from the global state object. Therefore, Feature Stores can be used for "Local Component State", which has the lifespan of a component. 

See an example in the [MiniRx Angular demo](https://angular-demo.mini-rx.io/#/counter).

#### Component Store
Component Stores live independently and are not related to something like a global state (e.g. when using @ngrx/store). 
The lifespan of a Component Store can be bound to a component ("Local Component State"), but it can also take the lifespan of the application.

#### Akita
The Akita Stores live independently next to each other. There is no real global state. You can use Akita Stores (which are destroyable too) for "Local Component State" following [this guide](https://datorama.github.io/akita/docs/angular/local-state) from the Akita docs.

### 4. Redux DevTools
#### MiniRx Feature Store
MiniRx can use Redux DevTools with the built-in [Redux DevTools Extension](https://mini-rx.io/docs/ext-redux-dev-tools). 
Every Feature Store state becomes part of the global state object, and it can be inspected with the Redux DevTools.

#### Component Store
There is no official solution for Redux DevTools with Component Store.

#### Akita
Akita has a [PlugIn for Redux DevTools support](https://datorama.github.io/akita/docs/enhancers/devtools). 
FYI: The separate Store states are merged into one big state object to make all state inspectable with the Redux DevTools. See the Akita DevTools source [here](https://github.com/datorama/akita/blob/master/libs/akita/src/lib/devtools.ts).

### 5. Cross-state selection
How can we select state from other store instances and pull that state into our current store (state service)?

#### MiniRx Feature Store
Every Feature Store state integrates into the global state object. Therefore, the corresponding feature states can be selected at anytime from the Redux `Store`(!) instance using `store.select`.
Alternatively you can use RxJS combination operators like `combineLatest` or `withLatestFrom` to combine state from other Feature Stores with state Observables of your current Feature Store.

#### Component Store
The Component Store `select` method also accepts a bunch of Observables to depend on (see docs [here](https://ngrx.io/guide/component-store/read#combining-selectors)). 
Of course these Observables can come from other services. Like this it is straight-forward to depend on (observable) state of other `ComponentStore` instances.

#### Akita
Akita has `combineQueries` to combine state from different `Query` instances. `combineQueries` is basically RxJS `combineLatest`. 
See the Akita combineQueries source [here](https://github.com/datorama/akita/blob/master/libs/akita/src/lib/combineQueries.ts).

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6asoiza0ckl0o62s35wc.jpeg)

### 6. Memoized Selectors
Memoized selectors can help to improve performance by reducing the number of computations of selected state. 
The selectors API (`createSelector`) is also great for Composition: Build selectors by combining existing selectors.

Examples for memoized selectors: 
- [NgRx Store selectors](https://ngrx.io/guide/store/selectors)
- [Redux Reselect](https://github.com/reduxjs/reselect)

#### MiniRx Feature Store
MiniRx comes with memoized selectors out-of-the-box. 
You can use the same `createFeatureSelector` and `createSelector` functions for the Redux `Store` API and for the `FeatureStore` API.

Read more in the [Feature Store memoized selectors documentation](https://mini-rx.io/docs/select-feature-state#memoized-selectors).

Example code of memoized selectors in the MiniRx Angular Demo: [Todos State Service](https://github.com/spierala/mini-rx-angular-demo/blob/main/src/app/modules/todo/state/todos-state.service.ts#L29)

#### Component Store
There is no official solution for Component Store. 
You could add @ngrx/store to use the memoized selectors, but it would probably be overkill to add the NgRx Redux Store just for that reason. Redux Reselect could be a better alternative.

#### Akita
No memoized selectors. You could most probably add Redux Reselect.

### 7. Effects
Effects are used to trigger side effects like API calls. 
We can also handle race-conditions more easily within an Effect by using RxJS flattening operators (`switchMap`, `mergeMap`, etc.).

#### MiniRx Feature Store
MiniRx Feature Store has Effects (https://mini-rx.io/docs/effects-for-feature-store).

FYI: Feature Store Effects have their equivalent in the Redux API of MiniRx: https://mini-rx.io/docs/effects

#### Component Store
Yes, there are Effects: https://ngrx.io/guide/component-store/effect

#### Akita
Yes, there are Effects: https://datorama.github.io/akita/docs/angular/effects. 
Effects come with a separate package (@datorama/akita-ng-effects).
The Effects API is not tied to a `Store` instance.

### 8. Undo
How can we undo state changes?

#### MiniRx Feature Store
MiniRx has the [UndoExtension](https://mini-rx.io/docs/ext-undo-extension) to support Undo of state changes. 
This is especially helpful if you want to undo optimistic updates (e.g. when an API call fails). Both the `FeatureStore` and the Redux `Store` API can undo specific state changes.
Feature Store exposes the `undo` method. 

Read more in the MiniRx docs: [Undo a setState Action](https://mini-rx.io/docs/update-feature-state#undo-setstate-actions-with-undo)

#### Component Store
No support for undo.

#### Akita
Akita has a State History PlugIn to undo state changes (https://datorama.github.io/akita/docs/plugins/state-history/). 
The API is much bigger than the one of Feature Store. But it seems to be difficult to undo a very specific state change (which is important when undoing optimistic updates).

### 9. Immutable State
Immutability is key when using state management: We only want to allow explicit state changes using the corresponding API (e.g. by using `setState`, `update` or by dispatching an Action in Redux). 
Mutating state however, might lead to unexpected behavior and bugs.
Immutable state helps to avoid such accidental state changes.

#### MiniRx Feature Store
MiniRx offers the [Immutable State Extension](https://mini-rx.io/docs/ext-immutable) to enforce immutable data. 
When the `ImmutableStateExtension` is added to the MiniRx Store both the Redux `Store` API and the `FeatureStore` API will use immutable data.
The Immutable State Extension "deepfreezes" the global state when state is updated. Mutating state will throw an exception.

#### Component Store
There is nothing in Component Store which can enforce immutability.

#### Akita
Akita "deepfreezes" the state object when state is updated (only in DEV mode). See the corresponding source code here: https://github.com/datorama/akita/blob/v6.2.0/libs/akita/src/lib/store.ts#L181

### 10. Framework-agnostic
#### MiniRx Feature Store
MiniRx is framework-agnostic. You can use MiniRx with any framework or even without framework.

See here the MiniRx Svelte Demo: https://github.com/spierala/mini-rx-svelte-demo

#### Component Store
Component Store is tied to Angular. Angular is a peer dependency in the [package.json](https://github.com/ngrx/platform/blob/13.0.1/modules/component-store/package.json#L25).

#### Akita
Akita is also framework-agnostic. You can see in this article how Svelte and Akita play together: [Supercharge Your Svelte State Management with Akita](https://netbasal.com/supercharge-your-svelte-state-management-with-akita-f1f9de5ef43d)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sg9baesmy5dexk6jh39m.jpeg)

Yes, you made it! I hope that you had fun to watch this fight!

## Conclusion
All competitors showed their skills, none of them went to the ground!

Who was your favorite?

Give it a star on GitHub: 

- ⭐ [MiniRx on GitHub](https://github.com/spierala/mini-rx-store)
- ⭐ [NgRx on GitHub](https://github.com/ngrx/platform)
- ⭐ [Akita on GitHub](https://github.com/datorama/akita)

## Notes

### What did we not cover?
For completeness, I want to list a few things that were out of scope for this fight: 

- Akita: EntityStore, Transactions, Akita Immer, Persist State, CLI
- Component Store: `updater` method, `tapResponse` operator

#### @rx-angular/state
Another cool lib which goes into the same direction as NgRx Component Store:
https://github.com/rx-angular/rx-angular/blob/master/libs/state/README.md

Maybe see you in the next fight! :)

## Thanks
#### Special thanks for reviewing this blog post:
- [Michael Rutzer - diePartments](https://twitter.com/diePartments)

#### Articles which inspired me:
- There was once a legendary fight in 2018, organized by [Orjan de Smet](https://twitter.com/orjandesmet): [NGRX VS. NGXS VS. AKITA VS. RXJS: FIGHT!](https://ordina-jworks.github.io/angular/2018/10/08/angular-state-management-comparison.html?utm_source=dormosheio&utm_campaign=dormosheio)

#### Photos:
- Photo by <a href="https://unsplash.com/@attentieattentie?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Attentie Attentie</a> on <a href="https://unsplash.com/s/photos/boxing-ring?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
- Photo by <a href="https://unsplash.com/@single_lens_reflex?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Dan Burton</a> on <a href="https://unsplash.com/s/photos/boxer-break?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
- Photo by <a href="https://unsplash.com/@matthewapayne?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Matthew Payne</a> on <a href="https://unsplash.com/s/photos/boxing-gloves?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
