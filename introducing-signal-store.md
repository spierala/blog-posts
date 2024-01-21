## Introducing MiniRx Signal Store

Welcome to Signal Store, the new state management library from MiniRx.

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

# API
## Redux

Redux is great to manage state at large scale. You get a clear separation of concerns, and you can easily split up your code:

- Actions: describe events with an optional payload
- Reducers: are pure functions which know how to update state (based on the current state and a given action)
- Effect: run side effects like API calls and handle race conditions (with RxJS flattening operators)
- Memoized selectors: functions which describe how to select state from the global state object

Your file structure can look like this:

![redux-file-structure.png](assets%2Fredux-file-structure.png)

### Memoized selectors
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

### Actions
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

### Reducer
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

### Effects
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

### Register reducers

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

### Component usage

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

### Redux DevTools

![devtools-redux-api.png](assets%2Fdevtools-redux-api.png)

## Feature Store

Feature Stores offer a more simple API to manage state, but they use Redux still under the hood.

A typical Feature Store looks like this:

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

### Redux DevTools

Feature Stores use Redux under the hood and their state becomes part of the global state object.

For that reason you can easily debug your Feature Stores with the Redux DevTools.

![devtools-feature-store-api.png](assets%2Fdevtools-feature-store-api.png)

### Advanced Feature Stores
When your state becomes more complex, Feature Store will scale with your state management needs.

- Use memoized selectors which are great for code re-use and performance
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

We have seen, that Feature Stores can be used to manage local component state. But Feature Stores integrate into the global state object and make use of the Redux Store internally.

With Component Stores you can manage state which should not become part of the global state object.

Furthermore, Component Stores can be used as a performance optimization if you have very frequent state updates or many store instances. 

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

You can guess it already... for more complex component states you can use memoized selectors (`createComponentStateSelector`) and the `rxEffect` method.