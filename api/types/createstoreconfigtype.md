# CreateStoreConfigType

{% hint style="info" %}
Interface of the zustand config function provided to [**middlewares**](createstoreoptionstype.md#custommiddlewares).
{% endhint %}

{% code title="src/types.ts" %}
```typescript
import type {
  State,
  GetState,
  SetState,
  StoreApi,
  StateCreator,
} from 'zustand'

type CreateStoreConfigType<
  StoreDataType,
  StoreApiType extends StoreApi<StoreType<StoreDataType>> = StoreApi<
    StoreType<StoreDataType>
  >
> = StateCreator<
  StoreType<StoreDataType>,
  SetState<StoreType<StoreDataType>>,
  GetState<StoreType<StoreDataType>>,
  StoreApiType
>
```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L32-L42" %}
