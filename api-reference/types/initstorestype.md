# InitStoresType

{% hint style="info" %}
Interface representing the data structure of the [**`initStores()`**](../initstores.md)method's output.
{% endhint %}

{% code title="src/types.ts" %}
```typescript
import type { EqualityChecker } from 'zustand'

type InitStoresType<StoresDataType> = {
  stores: {
    [StoreNameType in keyof StoresDataType]: CreateStoreType<
      StoresDataType[StoreNameType]
    >
  } & {
    rehydrate: () => Promise<boolean>
    reset: (options?: InitStoresResetOptionsType<StoresDataType>) => void
  }
  useStores: <StoreNameType extends keyof StoresDataType, Output>(
    storeName: StoreNameType,
    selector: (data: StoresDataType[StoreNameType]) => Output,
    equalityFn?: EqualityChecker<Output>
  ) => Output
}

```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L75-L89" %}

## API reference

### `stores`

```typescript
  stores: {
    [StoreNameType in keyof StoresDataType]: CreateStoreType<
      StoresDataType[StoreNameType]
    >
  } & {
    rehydrate: () => Promise<boolean>
    reset: (options?: InitStoresResetOptionsType<StoresDataType>) => void
  }
```

#### `stores.{storeName}`

You can directly access all the zustand stores you provided to `initStores()` via the `stores` key. Example:

```typescript
type StoresDataType = {
  nike: { name: string }
  fila: { name: string }
}

const nikeStore = createStore<StoresDataType['nike']>('nike', { name: 'Nike' })
const filaStore = createStore<StoresDataType['fila']>('fila', { name: 'fila' })

const { stores } = initStores<StoresDataType>([nikeStore, filaStore])

const nikeStoreName = stores.nike.getState().name // 'Nike'
const filaStoreName = stores.fila.getState().name // 'fila'
```

#### `stores.rehydrate`

Promise that allows you to manually rehydrate all the stores provided to `initStores()` that have the [**`persist`**](createstoreoptionstype.md#persist) middleware enabled. Example:

```typescript
type StoresDataType = {
  nike: { name: string }
  fila: { name: string }
}

const nikeStore = createStore<StoresDataType['nike']>('nike', { name: 'Nike' })
const filaStore = createStore<StoresDataType['fila']>('fila', { name: 'fila' })

const { stores } = initStores<StoresDataType>([nikeStore, filaStore])

await stores.rehydrate() // returns `true` upon completion
```

{% hint style="warning" %}
You shouldn't need to use this function as zustand stores rehydrate automatically:

* if you want to perform some actions upon rehydration, look into the [`onRehydrateStorage`](https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data#onrehydratestorage)`PersistOptions.`
* if you want to hide/show your app depending on the rehydration status, see the [**`<PersistGate />`**](../persistgate.md)component or the [**useRehydrate()**](../userehydrate.md) **** Hook.
{% endhint %}

#### `stores.reset`

Function that allows you to reset all (or part) of the stores you provided to `initStores()`. Example:

```typescript
type StoresDataType = {
  fb: { company: string }
  ggl: { company: string }
}

const fbStore = createStore<StoresDataType['fb']>('fb', { company: 'Facebook' })
const googleStore = createStore<StoresDataType['ggl']>('ggl', { company: 'Google' })

const { stores } = initStores<StoresDataType>([fbStore, googleStore])

stores.fb.getState().update(data => {
  data.company = 'Meta'
})

stores.ggl.getState().update(data => {
  data.company = 'Alphabet'
})

stores.reset({ omit: ['fb'] })

stores.fb.getState().data.company === 'Meta' // true, fbStore was omitted 
stores.ggl.getState().data.company === 'Google' // true, googleStore was reset

stores.reset()

stores.fb.getState().data.company === 'Meta' // false, fbStore was reset as well now
```

### `useStores`

```typescript
  useStores: <StoreNameType extends keyof StoresDataType, Output>(
    storeName: StoreNameType,
    selector: (data: StoresDataType[StoreNameType]) => Output,
    equalityFn?: EqualityChecker<Output>
  ) => Output
```

React Hook that allows you to consume the `data` from any of the store you provided to `initStores()`.

You'd use it exactly like you'd use a regular zustand store Hook, with the only difference that **`useStores()` expects the name of the store you want to get data from as its initial argument**. Example:

```typescript
import shallow from 'zustand/shallow'

type StoresDataType = {
  wishlist: { nextPurchase: string }
  friends: { best: string }
}

const storeA = createStore<StoresDataType['wishlist']>('wishlist', {
  nextPurchase: 'Watch',
})
const storeB = createStore<StoresDataType['friends']>('friends', {
  best: 'TBD',
})

const { useStores } = initStores<StoresDataType>([storeA, storeB])

const Component = () => {
  const bestFriendName = useStores('friends', data => data.best) // TBD
  const nextItemToPurchase = useStores(
    'wishlist',
    data => data.nextPurchase,
    shallow
  ) // 'Watch'
  return null
}
```

{% hint style="success" %}
`useStores()`really shines when you have a lot of stores in your app and don't want to have to import a lot of them/have a hard time remembering from which store you can get which data. You can now simply gather stores by topic/storage technology used/etc via `initStores().`
{% endhint %}
