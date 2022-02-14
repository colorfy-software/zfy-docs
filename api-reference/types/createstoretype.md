# CreateStoreType

{% hint style="info" %}
Interface representing the data structure of the[**`createStore()`**](../createstore.md)method's output.
{% endhint %}

{% code title="src/types.ts" %}
```typescript
import type { UseBoundStore } from 'zustand'
 import type {
  StoreApiWithPersist,
  StoreApiWithSubscribeWithSelector,
} from 'zustand/middleware'

type CreateStoreType<StoreDataType> = UseBoundStore<
  StoreType<StoreDataType>
> & {
  persist?: StoreApiWithPersist<StoreType<StoreDataType>>['persist']
  subscribeWithSelector?: StoreApiWithSubscribeWithSelector<
    StoreType<StoreDataType>
  >['subscribe']
}
```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L23-L30" %}

## API reference

{% hint style="info" %}
The following elements are provided on top of the **regular** ones returned by[ zustand's `createStore()`](https://github.com/pmndrs/zustand/blob/main/src/vanilla.ts).
{% endhint %}

### `persist`

```typescript
persist?: StoreApiWithPersist<StoreType<StoreDataType>>['persist']
```

{% hint style="warning" %}
Only provided if you've enabled the[**`persist`**](createstoreoptionstype.md#persist)middleware.
{% endhint %}

The detailed API reference is available here:

{% embed url="https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data#api" %}

### `subscribeWithSelector`

```typescript
subscribeWithSelector?: StoreApiWithSubscribeWithSelector<
  StoreType<StoreDataType>
>['subscribe']
```

{% hint style="warning" %}
Only provided if you've enabled the[**`subscribe`**](createstoreoptionstype.md#subscribe)middleware.
{% endhint %}

It uses the exact same API as the new [subscribeWithSelector()](https://github.com/pmndrs/zustand#using-subscribe-with-selector) middleware introduced with zustand [3.6.0](https://github.com/pmndrs/zustand/releases/tag/v3.6.0). The only difference is that zfy allows you to plug it in by simply adding a boolean to [**`createStore()`**](createstoreoptionstype.md#subscribe) **** 3rd (`options`) argument.

The detailed API reference is available here:

{% embed url="https://github.com/pmndrs/zustand/blob/main/src/middleware/subscribeWithSelector.ts" %}
