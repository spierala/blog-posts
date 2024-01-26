## Introducing MiniRx Signal Store

Welcome to MiniRx Signal Store, the new state management library from MiniRx.

* Signal Store is an **[Angular](https://angular.dev/)-only** state management library
* Signal Store **embraces [Angular Signals](https://angular.io/guide/signals)** and leverages **Modern Angular APIs** internally
* Signal Store is based on the same great concept as the original **[MiniRx Store](https://mini-rx.io/)**
    * Manage **global** state at large scale with the **Store (Redux) API**
    * Manage **global** state with a minimum of boilerplate using **Feature Stores**
    * Manage **local** component state with **Component Stores**
    * MiniRx always tries to find the sweet spot between powerful, simple and [lightweight](https://github.com/spierala/angular-state-management-comparison)
* Signal Store implements and promotes new **Angular best practices**:
    * **Signals** are used for **(synchronous) state**
    * **RxJS** is used for events and **asynchronous tasks**
* Signal Store helps to streamline your usage of [RxJS](https://rxjs.dev/) and [Signals](https://angular.io/guide/signals): e.g. `connect` and `rxEffect` understand both Signals and Observables
* Simple refactor: If you used MiniRx Store before, refactor to Signal Store will be pretty straight-forward: change the TypeScript imports, remove the Angular async pipes (and ugly non-null assertions (`!`)) from the template
* Signal Store has first-class support for OOP style (e.g. `MyStore extends FeatureStore`), but offers also functional creation methods (e.g. `createFeatureStore`)

#### Use-cases
MiniRx Signal Store is highly flexible and offers three different well-defined state containers out of the box:
- Store (Redux)
- FeatureStore
- ComponentStore

All three can be easily used together in your application. 
Depending on the use-case, you can choose the state container which suits your needs.

These are the typical use-cases:

![use-cases.png](assets%2Fuse-cases.png)

## MiniRx Signal Store APIs
Let's start with Redux... If you do not like Redux, do not run away! MiniRx Signal Store has first-class support for bypassing the infamous Redux boilerplate: Feature Stores.

### Redux introduction

The Redux pattern is great to manage state at large scale and MiniRx Signal Store offers a powerful Redux API.

Let's have a look at the basic building blocks of Redux:

- Actions: describe events with an optional payload
- Reducers: 
  - pure functions which know how to update state (based on the current state and a given action)
  - reducers are run for every action in order to calculate the next state
- Effects: listen to a specific action, run side effects like API calls and handle race conditions (with RxJS flattening operators)
- Memoized selectors: pure functions which describe how to select state from the global state object
- Store: holds the global state object, wires everything up (reducers, effects), offers the public API (`dispatch`, `select`)

You can see already, that these building blocks offer a nice separation of concerns. 
Every part does its own specialized thing, you can also split up your code more easily:

![redux-file-structure.png](assets%2Fredux-file-structure.png)

More benefits of Redux:
- Single source of truth for your state (state resides in **one** global state object)
- Uni-directional data flow
- Indirection of State and Actions
  - State: state is updated via actions, state changes are propagated via new Signal values
  - Actions: A single action can update state in one or many reducers, and trigger async tasks in effects
- Tooling: Redux DevTools

### Implementation

The Signal Store makes use of both Signals and Observables. Let's see how the Redux Store is implemented...

#### 🚦 Signals

- The global state is implemented as Angular Signal, because Signals are great for state! 
- `Store.select` returns a Signal
- Memoized selectors uses `computed` internally for memoization

#### Observables

- Actions are implemented as RxJS Subject, because Subjects are great for events!
- In Effects the Observable Actions stream is used to trigger side effects like API calls. You can easily handle race-conditions with RxJS flattening operators.

### Redux API

This is a just quick overview of the Signal Store Redux API. We mostly look at code examples to get a feeling for the Redux API...

#### Memoized selectors

Memoized selectors are used to select state from the global state object.

```ts
import {
  createFeatureStateSelector,
  createSelector,
} from '@mini-rx/signal-store';
import { Product } from '../models';

// State interface
export type ProductsState = {
  list: Product[];
}

// Memoized selectors
const getProductsFeature =
  createFeatureStateSelector<ProductsState>('product');
export const getProducts = createSelector(
  getProductsFeature,
  (state) => state.list,
);
```

#### Actions

You define Actions like this:

```ts
import { action, payload } from 'ts-action';
import { Product } from '../models';

export const loadProducts = action('[Products] load');
export const loadProductsSuccess = action(
  '[Products] load success',
  payload<Product[]>(),
);
export const loadProductsError = action('[Products] load error');

export const deleteProduct = action(
  '[Products] delete',
  payload<{ id: number }>(),
);
```
_FYI_ the powerful [ts-action](https://www.npmjs.com/package/ts-action) library is used to create actions with less boilerplate.

#### Reducer

Defining a reducer looks like this:

```ts
import { on, reducer } from 'ts-action';
import { ProductsState } from './index';
import {
  deleteProduct,
  loadProductsSuccess,
} from './product.actions';

const initialState: ProductsState = {
  list: [],
};

export const productReducer = reducer(
  initialState,
  on(loadProductsSuccess, (state, { payload }) => ({
    ...state,
    list: payload,
  })),
  on(deleteProduct, (state, { payload }) => ({
    ...state,
    list: state.list.filter((item) => item.id !== payload.id),
  })),
);
```
_FYI_ the powerful [ts-action](https://www.npmjs.com/package/ts-action) library is used to create reducers with less boilerplate.

#### Effects

You create an effect like this: 
- Listen to a specific action
- Run a (in most cases) asynchronous task
- Return a new action, when the task succeeded/failed

```ts
import { inject, Injectable } from '@angular/core';
import {
  Actions,
  createRxEffect,
  mapResponse,
} from '@mini-rx/signal-store';
import { ofType } from 'ts-action-operators';
import { ProductApiService } from '../product-api.service';
import { mergeMap } from 'rxjs';
import {
  loadProducts,
  loadProductsError,
  loadProductsSuccess,
} from './product.actions';

@Injectable()
export class ProductEffects {
  actions$ = inject(Actions);
  todosApi = inject(ProductApiService);

  loadTodos$ = createRxEffect(
    this.actions$.pipe(
      ofType(loadProducts),
      mergeMap(() =>
        this.todosApi.getTodos().pipe(
          mapResponse(
            (res) => loadProductsSuccess(res),
            (err) => loadProductsError,
          ),
        ),
      ),
    ),
  );
}
```

#### Register reducers

You can register reducers and effects with modern Angular standalone APIs (`provideStore`, `provideEffects`) when initializing the app:
```ts
import { ApplicationConfig } from '@angular/core';
import { routes } from './app.routes';
import {
  ImmutableStateExtension,
  provideEffects,
  provideStore,
  ReduxDevtoolsExtension,
} from '@mini-rx/signal-store';
import { productReducer } from './products/state/product.reducer';
import { ProductEffects } from './products/state/product.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore({
      reducers: {
        product: productReducer,
      },
      extensions: [
        new ReduxDevtoolsExtension({ name: 'Signal Store Demo' }),
        new ImmutableStateExtension(),
      ],
    }),
    provideEffects(ProductEffects),
  ],
};
```

#### Lazy load reducers

It is possible to register reducers together with a lazy loaded module (see `provideFeature`).
For registering effects you can use `provideEffects`.

```ts
import { Routes } from '@angular/router';
import { ProductShellComponent } from './products-shell/product-shell.component';
import { productReducer } from './state/product.reducer';
import {
  provideEffects,
  provideFeature,
} from '@mini-rx/signal-store';
import { ProductEffects } from './state/product.effects';

export const productRoutes: Routes = [
  {
    path: '',
    component: ProductShellComponent,
    // Lazy load the products state
    providers: [
      provideFeature('products', productReducer),
      provideEffects(ProductEffects),
    ],
  },
];
```

#### Component usage

Your components can read state from the store via the `select` method. `select` returns an Angular Signal.

The `dispatch` method is used to notify the store about new Events (Actions).
```ts
import { Component, inject, OnInit, Signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Store } from '@mini-rx/signal-store';
import { Product } from '../models';
import { getProducts } from '../state';
import {
  deleteProduct,
  loadProducts,
} from '../state/product.actions';

@Component({
// ...
})
export class ProductShellComponent implements OnInit {
  private store = inject(Store);
  products: Signal<Product[]> = this.store.select(getProducts);

  ngOnInit() {
    this.store.dispatch(loadProducts());
  }

  deleteProduct(todo: Product) {
    this.store.dispatch(deleteProduct({ id: todo.id }));
  }
}
```

#### Redux DevTools

Of course, MinRx Signal Store supports Redux DevTools. With Redux DevTools you can inspect the current state and see which actions have been dispatched.

![devtools-redux-api.png](assets%2Fdevtools-redux-api.png)

#### What else

And there is more which we haven't covered: 
- Meta reducers
- We can register extensions: Redux DevTools, Immutable Extension, Undo Extension, Logger Extension
- You can write your own extensions

## Feature Store

Feature Store offers a more simple API to manage state, but it still uses the Redux store under the hood.

### Implementation
And again, MiniRx implements and promotes new Angular best practices also in the Feature Store:
- State is implemented and exposed as Angular Signal
- RxJS Subject is used for effects to run asynchronous tasks

### What's included
- `setState` update feature state directly with a minimum of boilerplate
- `select` read feature state as Angular Signal
- `rxEffect` trigger side effects like API calls and handle race-conditions with RxJS flattening operators
  - effects can be triggered with Signals, Observables and Raw Values
- `connect` connect external sources like Signals or Observables to your feature state
- `undo` undo state changes (requires the Undo extension)

### Example using `extends`

A typical Feature Store looks like this (a Singleton Angular service which extends `FeatureStore`):

```ts
import { inject, Injectable, Signal } from '@angular/core';
import { FeatureStore } from '@mini-rx/signal-store';
import { Todo } from './models';
import { TodoApiService } from './todos-api.service';

type TodoState = {
  list: Todo[];
};

const initialState: TodoState = {
  list: [],
};

@Injectable({
  providedIn: 'root',
})
export class TodosStoreService extends FeatureStore<TodoState> {
  private api = inject(TodoApiService);

  todosDone: Signal<Todo[]> = this.select((state) =>
    state.list.filter((item) => item.isDone),
  );
  todosNotDone: Signal<Todo[]> = this.select((state) =>
    state.list.filter((item) => !item.isDone),
  );

  constructor() {
    super('todo', initialState);
  }

  loadTodos(): void {
    this.api
      .getTodos()
      .subscribe((todos) => this.setState({ list: todos }));
  }

  toggleDone(todo: Todo): void {
    this.setState((state) => ({
      list: state.list.map((item) =>
        item.id === todo.id
          ? { ...item, isDone: !item.isDone }
          : item,
      ),
    }));
  }
}
```

You can see that state is read via the `select` method. Instead of dispatching actions you use `setState` to update state directly with a minimum of boilerplate.

### Feature Store and Redux DevTools

Feature Stores use Redux under the hood and their state becomes part of the global state object.

For that reason you can easily debug your Feature Stores with the Redux DevTools.

![devtools-feature-store-api.png](assets%2Fdevtools-feature-store-api.png)

### Advanced Feature Stores
When your state becomes more complex, Feature Store will scale with your state management needs.

- Use memoized selectors (`createFeatureStateSelector`, `createSelector`) which are great for code re-use and performance
  - Memoized selectors can easily be moved to another file
- Use `rxEffect` to trigger side effects like API calls and handle race conditions (e.g. with RxJS `switchMap`)

```ts
import { inject, Injectable, Signal } from '@angular/core';
import {
  createFeatureStateSelector,
  createSelector,
  FeatureStore,
  tapResponse,
} from '@mini-rx/signal-store';
import { Todo } from './models';
import { TodoApiService } from './todos-api.service';
import { switchMap } from 'rxjs';

// Memoized Selectors
const getFeatureState = createFeatureStateSelector<TodoState>();
const getList = createSelector(getFeatureState, (state) => state.list);
const getTodosDone = createSelector(getList, (list) =>
  list.filter((item) => item.isDone),
);
const getTodosNotDone = createSelector(getList, (list) =>
  list.filter((item) => item.isDone),
);

@Injectable({
  providedIn: 'root',
})
export class TodosStoreService extends FeatureStore<TodoState> {
  private api = inject(TodoApiService);

  todosDone: Signal<Todo[]> = this.select(getTodosDone);
  todosNotDone: Signal<Todo[]> = this.select(getTodosNotDone);

  // Create an Effect
  loadTodos = this.rxEffect<void>(
    switchMap(() =>
      this.api.getTodos().pipe(
        tapResponse({
          next: (todos) => this.setState({ list: todos }),
          error: (err) => console.log(err),
        }),
      ),
    ),
  );
}
```

### Manage Component State with Feature Stores

You can easily create Feature Stores which are bound to the component life-cycle.

Simply create a Feature Store inside your component.

This example uses the functional creation method `createFeatureStore` which creates a new Feature Store instance for us.

```ts
@Component({
// ...
})
export class TodosShellComponent implements OnInit {
  private api = inject(TodoApiService);

  todoStore = createFeatureStore('todo', initialState);

  todosDone: Signal<Todo[]> = this.todoStore.select((state) =>
    state.list.filter((item) => item.isDone),
  );
  todosNotDone: Signal<Todo[]> = this.todoStore.select((state) =>
    state.list.filter((item) => !item.isDone),
  );

  ngOnInit(): void {
    this.loadTodos();
  }

  loadTodos() {
    this.api.getTodos().subscribe((todos) => this.todoStore.setState({ list: todos }));
  }
}
```
The "todo" Feature Store will be created and destroyed together with the component.

In the Redux DevTools you can see that the "todo" Feature Store had been created and destroyed.

Create:

![devtools-feature-store-api--init.png](assets%2Fdevtools-feature-store-api--init.png)

Destroy:

![devtools-feature-store-api--destroy.png](assets%2Fdevtools-feature-store-api--destroy.png)

## Component Store

We have just seen, how Feature Stores can be used to manage local component state. But Feature Stores integrate into the global state object and make use of the Redux Store internally.

With Component Stores you can manage state which should not become part of the global state object.

Furthermore, Component Stores can be used as a performance optimization if you have very frequent state updates or many store instances. 

Component Store has the same API as Feature Store. Refactoring from Component Store to Feature Store and vice versa means changing two lines of code.
```ts
@Component({
// ...
})
export class TodosShellComponent implements OnInit {
  private api = inject(TodoApiService);

  // Using Component Store!
  // We do not need a feature key
  todoStore = createComponentStore(initialState);

  todosDone: Signal<Todo[]> = this.todoStore.select((state) =>
          state.list.filter((item) => item.isDone),
  );
  todosNotDone: Signal<Todo[]> = this.todoStore.select((state) =>
          state.list.filter((item) => !item.isDone),
  );

  ngOnInit(): void {
    this.loadTodos();
  }

  loadTodos() {
    this.api.getTodos().subscribe((todos) => this.todoStore.setState({ list: todos }));
  }
}
```
### Advanced Component Stores

You can guess it already... for more complex component states you can use memoized selectors (`createComponentStateSelector`, `createSelector`) and the `rxEffect` method.

## RxJS and Signal Interop

In modern Angular Observables and Signals will coexist.
Therefore, modern Angular state management should help you to streamline the usage of Observables and Signals.
That is exactly what MiniRx does in the Feature Store and Component Store APIs.

### `rxEffect`

`rxEffect` is used to trigger side effects like API calls.
There are three different ways to trigger the side effect:

- Raw value
- Signal
- Observable

The example below listens to Signal changes in order to fetch new data.

In Angular 17.1 we have Signal Inputs. You can use a Signal (Input) to fetch the component data:

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

Alternatively, an Observable or Raw value could be used to trigger the API call.

### `connect`

With `connect` you can connect your store with external sources like Observables and Signals.
This helps to make your store the Single Source of Truth for your state.

```ts
import { Component, signal } from '@angular/core';
import { ComponentStore, createComponentStore } from '@mini-rx/signal-store';
import { timer } from 'rxjs';

interface State {
  counterFromObservable: number;
  counterFromSignal: number;
}

@Component({
// ...
})
export class ConnectDemoComponent {
  cs: ComponentStore<State> = createComponentStore<State>({
    counterFromObservable: 0,
    counterFromSignal: 0,
  });

  constructor() {
    const interval = 1000;

    const observableCounter$ = timer(0, interval); // Observable
    const signalCounter = signal(0); // Signal

    // Connect external sources (Observables or Signals) to the Component Store
    this.cs.connect({
      counterFromObservable: observableCounter$, // Observable
      counterFromSignal: signalCounter, // Signal
    });

    setInterval(() => signalCounter.update((v) => v + 1), interval);
  }
}
```