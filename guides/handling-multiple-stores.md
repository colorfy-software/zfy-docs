# Handling multiple stores

Zustand's philosophy is to be a hassle-free, unopinionated state management library. You create your store, add some data & logic and use it anywhere as a React Hook (or via the vanilla API). Then the more your app grows, the more you might have different stores. This happens until you reach the point where you might be lost in the maze you lost yourself in. "_Which store do I have to import to get this piece of data again?_". "_Which storage did I use for this store?_". "_Wait...which stores do I reset when the user logs out?_".

In order to address all of these questions - and because these were situations we've experienced way too often ourselves - we decided to come up with a new API to alleviate some of that cognitive load: [**`initStores()`**](../api-reference/initstores.md).

## InitStores()

This new method allows you to bundle stores together. You can batch stores based on the storage solution used, on usage context inside your app or just store name length. What are the criteria is totally up to you. You can even have multiple `initStores()` in your app, it all depends on your needs.

`initStores()` outputs an object that contains 2 things: [`stores`](handling-multiple-stores.md#undefined) and [`useStores()`](handling-multiple-stores.md#undefined).

```typescript
import { createStore, initStores } from '@colorfy-software/zfy'

type StoresDataType = {
  user: { followers: number }
  settings: { mode: 'light' | 'dark' | 'automatic' }
}

const initialDataUser: StoresDataType['user'] = { followers: 0 }
const initialDataSettings: StoresDataType['settings'] = { mode: 'light'}

const storeA = createStore<StoresDataType['user']>('user', initialDataUser)
const storeB = createStore<StoresDataType['settings']>(
  'settings',
  initialDataSettings
)

const { stores, useStores } = initStores<StoresDataType>([storeA, storeB])
```

### `stores`

This object contains all the stores you provided to `initStore()`, accessible for future use via the store name you provided to their [**`createStore()`**](../api-reference/createstore.md). For instance, if we continue with the `stores` created in the previous snippet, you're now able to use each store individually as is:

```typescript
  stores.user.getState().update(data => {
    data.followers += 1
  })
  
  stores.settings.getState().update(data => {
    data.mode = 'dark'
  })
```

In short, you can expect these to work the same as any other zustand store, with the only difference being that you can access several of them from the same place now.

On top of that, `stores` also exposes 2 fairly useful methods: [**`stores.rehydrate()`**](../api-reference/types/initstorestype.md#stores.rehydrate) & [**`stores.reset()`**](../api-reference/types/initstorestype.md#stores.reset).

#### `stores.rehydrate()`

{% hint style="info" %}
If you use[**`<PersistGate/>`**](../api-reference/persistgate.md)or[**`useRehydrate()`**](../api-reference/userehydrate.md), you should not need this method as they automatically take care of the rehydration process for you.
{% endhint %}

Otherwise, you can manually rehydrate all stores:

```typescript
const isRehydrated = await stores.rehydrate()
```

{% hint style="info" %}
If you provided a store that's not persisted, zfy will simply skip it and continue until it finds one to rehydrate or just resolves`true`.
{% endhint %}

#### `stores.reset()`

This method lets you reset all (or some) of your stores in one go. This can be really handy if you have to clear multiple stores at once on a given user action, ie: a logout.

```typescript
// resets both stores
stores.reset()

// only resets the user store, settings will stay untouched
stores.reset({ omit: ['settings'] })
```

### `useStores()`

This React Hook strieves to achieve the same goal as `stores`. With it you can now consume multiple store Hooks via a single source, without having to bother about loads and loads of `import` statement:

```typescript
const App = () => {
  const amountOfFollowers = useStores('user', data => data.followers)
  const settings = useStores(
    'settings',
    data => data,
    (prevData, newData) => prevData.mode === newData.mode
  )
  return null
}
```

You'll notice that `useStores` can be used like any zustand store Hook with a selector and even an equality function. **The only difference is that it requires you to indicate which store to use as its first argument**. Everything else is identical.

{% hint style="info" %}
You can look into the [**`InitStoreType`**](../api-reference/types/initstorestype.md) API reference for a more in-depth explanation of the inner workings of [**`initStores()`**](../api-reference/initstores.md).
{% endhint %}
