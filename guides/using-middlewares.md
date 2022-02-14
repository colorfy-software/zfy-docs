# Using middlewares

By default, a zustand store only lets you get and set data. As soon as you'll want it to do more (persist data, log updates, subscribe to changes, etc) is when zustand middlewares will come into play. zfy offers 2 types of middleware for that purpose: [built-in](using-middlewares.md#built-in) and [custom](using-middlewares.md#undefined).

## Built-in

Out of the box, you have access to a [**`persist`**](../api/types/createstoreoptionstype.md#persist), [**`log`**](../api/types/createstoreoptionstype.md#log) and [**`subscribe`**](../api/types/createstoreoptionstype.md#subscribe) middlewares. There's not much you'll need to do here as these middlewares can be configured directly in [**`createStore()`**](../api/createstore.md). You can use none, one or all of them at once:

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage'
import { createStore } from '@colorfy-software/zfy'

const data = { name: 'zfy' }

type StoreDataType = typeof data

const store = createStore<StoreDataType>('store', data, {
  persist: { getStorage: () => AsyncStorage },
  subscribe: true,
  log: true,
})

const listener = (newName, oldName) => console.log({ newName, oldName })

const unsubscribe = store.subscribeWithSelector?.(
  state => state.data.name,
  listener,
  { fireImmediately: true }
)
```

Here, in just 3 lines, you've enabled zustand's own persist & subscribeWithSelector middlewares, on top of zfy's [**`log`**](../api/types/createstoreoptionstype.md#log) middleware.&#x20;

{% hint style="info" %}
* The [**`log`**](../api/types/createstoreoptionstype.md#log) middleware will print the store name, previous state, payload data and new state upon each update.
* You can refer to zustand's documentation for details about [persist](https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data) & [subscribeWithSelector](https://github.com/pmndrs/zustand#using-subscribe-with-selector) middlewares.
{% endhint %}

## Custom

At some point, your use cases might require more than what the 3 built-in middlewares can offer. It could be that you want to use another of zustand's middleware or even plug in your very own custom middleware. zfy allows you to do so via the [**`customMiddlewares`**](../api/types/createstoreoptionstype.md#custommiddlewares) option:

```typescript
import { devtools } from 'zustand/middleware'
import { ZfyMiddlewareType, createStore } from '@colorfy-software/zfy'

const data = { name: 'zfy' }

type StoreDataType = typeof data

const customMiddleware: ZfyMiddlewareType<StoreDataType> =
  (storeName, config) => (set, get, api) =>
    config(
      (args) => {
        // 📍Do anything in your middlware
        set(args)
        console.log(storeName)
      },
      get,
      api
    )

const zustandMiddleware: ZfyMiddlewareType<StoreDataType> = (storeName, config) =>
  devtools((set, get, api) => config(set, get, api), { name: storeName })

const store = createStore('store', data, {
  customMiddlewares: [customMiddleware, zustandMiddleware],
})
```

{% hint style="warning" %}
This snippet is only provided as an example to show you how to integrate a zustand middleware. You shouldn't rely on `devtools` as zfy uses Immer instead of Redux-like actions & dispatchers.
{% endhint %}

{% hint style="success" %}
You'll notice that zfy's middlewares look a tiny bit different from zustand's. Instead of only `config` as in the initial function argument, you also receive the`storeName` (the one you provided as [**`createStore()`**](../api/createstore.md)first argument). **This can be very useful if you want to have conditional logic in your middleware based on which store is using it for instance**.
{% endhint %}
