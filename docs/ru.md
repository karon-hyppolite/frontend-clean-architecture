# Frontend Clean Architecture

Пример приложения на React и TypeScript, который показывает Clean Architecture в frontend-кодовой базе с функциональным уклоном.

## Назначение

Этот проект показывает один из способов разделить frontend-приложение на доменный, прикладной и инфраструктурный слои, сохранив за React роль UI и адаптеров.

Репозиторий можно использовать как небольшой справочный пример для:

- моделирования доменных сущностей и правил вне компонентов;
- изоляции юз-кейсов от логики представления;
- подключения сервисов через хуки и адаптеры;
- обсуждения компромиссов в прагматичной, а не строго каноничной реализации.

## Структура проекта

- `src/domain`: бизнес-сущности и доменные правила.
- `src/application`: юз-кейсы и прикладная оркестрация.
- `src/services`: инфраструктура и интеграционные детали.
- `src/ui`: React-компоненты и логика, ориентированная на представление.
- `src/shared-kernel.d.ts`: общие глобальные типы, используемые в разных модулях.

## Что стоит учитывать

В репозитории есть несколько намеренных упрощений и компромиссов, чтобы пример оставался компактным и сфокусированным.

### Shared Kernel

Shared Kernel содержит код и данные, от которых могут зависеть разные модули, но только если такая зависимость не создаёт лишнего зацепления.

В этом проекте shared kernel в основном состоит из глобальных определений типов, собранных в `src/shared-kernel.d.ts`.

### Зависимость в домене

Функция `createOrder` в `src/domain/order.ts` использует `currentDatetime`, чтобы установить время создания заказа. Строго говоря, это создаёт зависимость домена от внешнего вспомогательного кода, а это не лучший вариант.

Более чистый подход состоял бы в том, чтобы создавать дату вне доменной сущности и передавать её явно:

```ts
async function orderProducts(user: User, { products }: Cart) {
  const datetime = currentDatetime();
  const order = new Order(user, products, datetime);

  // ...
}
```

### Тестируемость юз-кейса

Логика `orderProducts` в `src/application/orderProducts.ts` сейчас обёрнута в React-ориентированную форму, поэтому изолированное тестирование получается менее прямолинейным, чем могло бы быть.

В более строгом варианте юз-кейс был бы отдельной функцией, а зависимости передавались бы явно:

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

Тогда хук выполнял бы только роль адаптера:

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

Здесь эта дополнительная структура намеренно опущена, чтобы пример оставался сосредоточен на основных архитектурных идеях.

### Ручное внедрение зависимостей

В прикладном слое сервисы внедряются вручную:

```ts
export function useAuthenticate() {
  const storage: UserStorageService = useUserStorage();
  const auth: AuthenticationService = useAuth();

  // ...
}
```

В более крупной системе это можно было бы формализовать через dependency injection. В этой кодовой базе React-хуки фактически выступают в роли лёгкого контейнера, который предоставляет реализации нужных интерфейсов.

Такой выбор делает пример проще для чтения, даже если он менее каноничен, чем полноценная DI-настройка.
