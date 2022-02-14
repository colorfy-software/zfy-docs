# ZfyMiddlewareType

{% hint style="info" %}
Interface representing the data structure of a zfy [**customMiddleware**](createstoreoptionstype.md#custommiddlewares).
{% endhint %}

{% code title="src/types.ts" %}
```typescript
import type { StoreApi } from 'zustand'

export type ZfyMiddlewareType<
  StoreDataType,
  StoreApiType extends StoreApi<StoreType<StoreDataType>> = StoreApi<
    StoreType<StoreDataType>
  >
> = (
  storeName: string,
  config: CreateStoreConfigType<StoreDataType>,
  options?: CreateStoreOptionsType<StoreDataType>
) => CreateStoreConfigType<StoreDataType, StoreApiType>
```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L60-L69" %}
