The [Angular](https://angular.dev/) renaissance is still ongoing. [MiniRx](https://github.com/spierala/mini-rx-store) is part of that renaissance and released a new Signal-based state management library for Angular: **[MiniRx Signal Store](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md)**.

* [MiniRx Signal Store on GitHub](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md)
* [MiniRx Signal Store on NPM](https://www.npmjs.com/package/@mini-rx/signal-store)

There are many reasons why MiniRx Signal Store is a great state management solution for Angular..., but these are the top ten reasons:

1. **All-in-one solution**: With MiniRx Signal Store you get three well-defined state containers out of the box.
   * Manage **global** state at large scale with the **[Store (Redux) API](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md#redux-api)**
   * Manage **global** state with a minimum of boilerplate using **[Feature Stores](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md#feature-store-api)**
   * Manage **local** component state with **[Component Stores](https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md#component-store-api)**
2. **Flexibility**: All three state containers can be easily used together in your application. Depending on the use-case, you can choose the state container which suits your needs.
   ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3fpe701m2xa80s7c7c0o.png)
3. **Highly integrated**: MiniRx Signal Store was designed as an all-in-one solution from the beginning. This allows for a high level of integration:
   * Feature Store uses the Redux Store under the hood
   * Feature Store state integrates into the global state object of the Redux Store
   * Feature Store automatically uses all the extensions of the Redux Store (e.g. Redux DevTools extension, ImmutableState extension)
   * Feature Store and Component Store share the same API
4. **Made for modern Angular**: MiniRx Signal Store uses modern Angular APIs (e.g. Signal, DestroyRef) and offers deep integration with Angular.
5. **New Angular best practices:** MiniRx Signal Store implements and promotes new Angular best practices:
   * [Signals](https://angular.dev/guide/signals) are used for (synchronous) state
   * [RxJS](https://rxjs.dev/) is used for events and asynchronous tasks
6. **RxJS and Signals interop**: MiniRx Signal Store streamlines your usage of RxJS and Signals: e.g. `connect` and `rxEffect` understand both Signals and Observables
7. [**Lightweight**](https://github.com/spierala/angular-state-management-comparison?tab=readme-ov-file#state-management-bundle-size-comparison-angular)
8. **Extendable**: MiniRx Signal Store comes with powerful extensions:
   * Immutable State Extension: Make your Signal state immutable to prevent mutating state accidentally.
   * Redux DevTools Extension: inspect state and state changes at anytime in the Redux DevTools (available for the Redux Store and Feature Store).
   * Undo Extension: Undo state changes.
   * Logger Extension: Log actions and updated state in the JS console.
   * You can easily create your own extensions!
9. **OOP-style**: Your fellow fullstack Angular developers, who are used to object-oriented programming (Java, .NET, ...) will love MiniRx Signal Store. MiniRx Signal Store exposes TypeScript classes which can be extended or used with `new`.
10. **Framework-agnostic code**: Although MiniRx Signal Store is an Angular library, your state management code is almost framework-agnostic. Signals are an internal implementation detail of the Signal Store. Therefore, you can easily refactor your state management layer to the original (RxJS-based) [MiniRx Store](https://mini-rx.io/) and use it in whatever framework you want (e.g. Svelte).

### You can try MiniRx Signal Store today

`npm i @mini-rx/signal-store`

Documentation: https://github.com/spierala/mini-rx-store/blob/master/libs/signal-store/README.md

### Do you like MiniRx Signal Store?

Give it a star on GitHub:
⭐ [MiniRx platform on GitHub](https://github.com/spierala/mini-rx-store)

Thank you!

## Thanks
Special thanks for reviewing this blog post:

- [Pieter van Poyer](https://github.com/PieterVanPoyer)