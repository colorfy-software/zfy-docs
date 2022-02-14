# CreateStoreOptionsType

{% hint style="info" %}
Interface of the 3rd (`options)`argument accepted by the [**`createStore()`**](../createstore.md) method.
{% endhint %}

{% code title="src/types.ts" %}
```typescript
import type { PersistOptions } from 'zustand/middleware'

interface CreateStoreOptionsType<StoreDataType> {
  log?: boolean
  subscribe?: boolean
  persist?: Omit<
    PersistOptions<StoreType<StoreDataType>>,
    'name' | 'blacklist' | 'whitelist'
  > & {
    name?: string
    getStorage: Exclude<
      PersistOptions<StoreType<StoreDataType>>['getStorage'],
      undefined
    >
  }
  customMiddlewares?: ZfyMiddlewareType<StoreDataType>[]
}
```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L44-L58" %}

## API reference

### `log`

```typescript
log?: boolean
```

Setting this flag will enable/disable the custom logger middleware. The logger highlights the previous state, the payload of the update and the content of the new state.

{% hint style="warning" %}
There is a known bug if you're using Flipper as it doesn't allow syntax highlighting for logs ([yet](https://github.com/facebook/flipper/issues/3173)).
{% endhint %}

### `subscribe`

```typescript
subscribe?: boolean
```

Setting this flag will enable/disable the new [`subscribeWithSelector()`](https://github.com/pmndrs/zustand#using-subscribe-with-selector) middleware and make it accessible via [**`createStore().subscribeWithSelector()`**](createstoretype.md#subscribewithselector).

### `persist`

```typescript
  persist?: Omit<
    PersistOptions<StoreType<StoreDataType>>,
    'name' | 'blacklist' | 'whitelist'
  > & {
    name?: string
    getStorage: Exclude<
      PersistOptions<StoreType<StoreDataType>>['getStorage'],
      undefined
    >
  }
```

It uses the same `PersistOptions` as zustand's persist middleware with only 2 differences:

* <mark style="color:red;">**`getStorage`**</mark><mark style="color:red;">** **</mark><mark style="color:red;">**is now required instead of optional.**</mark> <mark style="color:red;"></mark><mark style="color:red;"></mark> This is due to the fact that zustand uses LocalStorage by default when this field is not provided, which wouldn't work with React Native.
* **`name` is now optional instead of required**. This is because zfy simply uses the `name` you provided as [**`createStore()`**](../../guides/creating-and-using-a-store.md) **** 1st argument by default. Of course, you can still override it here if need be.

The detailed API reference is available here:

{% embed url="https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data#options" %}

### `customMiddlewares`

```typescript
customMiddlewares?: ZfyMiddlewareType<StoreDataType>[]
```

This field allows you to provide middlewares to zfy. This could be one that you created yourself from scratch or a prebuilt zustand middleware that zfy doesn't already expose. Please refer to the [**Using middlewares**](../../guides/using-middlewares.md#custom) guide if you want to know how to use this option.
