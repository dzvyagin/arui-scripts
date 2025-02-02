@alfalab/client-event-bus

Библиотека позволяет обмениваться данными в приложениях с модульной архитектурой (module federation), используя событийную модель браузера.
Это особенно актуально для приложений на базе `Webpack Module Federation`, обеспечивая взаимодействие без создания жестких зависимостей.

По сути представляет собой [`EventEmitter`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) с добавлением методов для
получения последнего отправленного события уже после того, как это событие произошло.

Для того чтобы добавить типизацию событий на проекте, не трогая основной пакет, можно сделать следующее:

`constants/event-bus.ts` - заводим ключ, по которому создается шина данных
```ts
// constants/event-bus.ts
export const BUS_KEY = 'my-first-bus';
```

`types/event-bus.ts` - определяем типы событий
```ts
// types/event-bus.ts
type EventType = 'busValueFirst' | 'busValueSecond'
type EventPayload = string | null

export type EventTypes = Record<EventType, EventPayload>;
```

`types/event-types.d.ts` - добавляем файл для типизации функций `getEventBus` и `createBus`
```ts
// types/event-types.d.ts
import type { AbstractAppEventBus } from '@alfalab/client-event-bus';

import { BUS_KEY } from '~/constants/event-bus';
import type { EventTypes } from '~types/event-bus'

export declare type EventBus = AbstractAppEventBus<EventTypes>;

declare module '@alfalab/client-event-bus' {
    export declare function getEventBus(busKey: typeof BUS_KEY): EventBus;
}

declare module '@alfalab/client-event-bus' {
    export declare function createBus(
        key: typeof BUS_KEY,
        params?: EventBusParams,
    ): EventBus;
}
```

## Рекомендации использования

Для именования событий предлагается следующие договоренности:

- Имя события начинается с названия вашего проекта, исключая префикс вашей системы/направления (`corp-`, `ufr-` и так далее) и суффикс `-ui`. То есть если ваш проект называется `ufr-cards-ui` событие должно начинаться с `cards_`, `corp-sales-ui` это соответственно `sales_`.
- Название события пишется в `camelCase`.

## Возвращаемое значение

Возвращает `EventBus`, со следующими методами:

- `addEventListener(eventName: string, eventHandler: (event: CustomEvent) => void, options?: AddEventListenerOptions)` - [стандартная функция](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) добавления подписки на событие
- `removeEventListener(eventName: string, eventHandler: (event: CustomEvent) => void, options?: EventListenerOptions)` - [стандартная функция](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener) удаления подписки на событие
- `getLastEventDetail(eventName: string)` - Функция, которая возвращает последнее событие заданного типа. Если событие еще не происходило - возвращает `undefined`
- `addEventListenerAndGetLast(eventName: string, eventHandler: (event: CustomEvent) => void, options?: AddEventListenerOptions)` - объединяет в себе `addEventListener` и `getLastEvent`. Подписывает на событие и возвращает последнее событие этого типа

## Использование в react
Если вам нужно использовать значение из event-bus в react коде - вы можете использовать хук `useEventBusValue`:
```tsx
import { useEventBusValue } from '@alfalab/client-event-bus';

const MyComponent = () => {
    const currentOrganizationId = useEventBusValue('shared_currentOrganizationId');

    return (
        <div>
            ID текущей организации: {currentOrganizationId}
        </div>
    )
}
```

Хук всегда будет возвращать последнее значение из eventBus. При изменениях значения будет происходить ререндер.
