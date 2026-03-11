# Frontend Clean Architecture

A React + TypeScript example app that demonstrates Clean Architecture in a functional-style frontend codebase.

## Purpose

This project shows one way to separate a frontend application into domain, application, and infrastructure concerns while keeping React focused on UI and adapters.

The repository is useful as a small reference for:

- modeling domain entities and rules outside components;
- isolating use cases from view logic;
- wiring services through hooks and adapters;
- discussing tradeoffs in a pragmatic, non-purist implementation.

## Project Structure

- `src/domain`: business entities and domain rules.
- `src/application`: use cases and application-level orchestration.
- `src/services`: infrastructure and integration details.
- `src/ui`: React components and view-facing logic.
- `src/shared-kernel.d.ts`: shared global types used across modules.

## Things to Consider

This repository intentionally contains a few compromises and simplifications so the example stays small and focused.

### Shared Kernel

The shared kernel contains code and data that any module may use, but only when that dependency does not create unnecessary coupling.

In this project, the shared kernel is mostly global type definitions collected in `src/shared-kernel.d.ts`.

### Dependency in the Domain

The `createOrder` function in `src/domain/order.ts` uses `currentDatetime` to set the order creation time. Strictly speaking, that makes the domain depend on an external helper, which is not ideal.

A cleaner approach would create the date outside the domain entity and pass it in explicitly:

```ts
async function orderProducts(user: User, { products }: Cart) {
  const datetime = currentDatetime();
  const order = new Order(user, products, datetime);

  // ...
}
```

### Use Case Testability

The `orderProducts` logic in `src/application/orderProducts.ts` is currently wrapped in a React-oriented shape, which makes isolated testing less direct than it could be.

In a stricter setup, the use case would be a standalone function and dependencies would be passed in explicitly:

```ts
type Dependencies = {
  notifier?: NotificationService;
  payment?: PaymentService;
  orderStorage?: OrderStorageService;
};

async function orderProducts(
  user: User,
  cart: Cart,
  dependencies: Dependencies = defaultDependencies
) {
  const { notifier, payment, orderStorage } = dependencies;

  // ...
}
```

The hook would then act only as an adapter:

```ts
function useOrderProducts() {
  const notifier = useNotifier();
  const payment = usePayment();
  const orderStorage = useOrdersStorage();

  return (user: User, cart: Cart) =>
    orderProducts(user, cart, {
      notifier,
      payment,
      orderStorage,
    });
}
```

That extra structure was intentionally skipped here to keep the example centered on the main architectural ideas.

### Manual Dependency Injection

In the application layer, services are injected by hand:

```ts
export function useAuthenticate() {
  const storage: UserStorageService = useUserStorage();
  const auth: AuthenticationService = useAuth();

  // ...
}
```

In a larger system, this could be formalized through dependency injection. In this codebase, React hooks effectively act as a lightweight container that provides implementations for the required interfaces.

That choice keeps the example easier to read, even if it is less canonical than a full DI setup.
