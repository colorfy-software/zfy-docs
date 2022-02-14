# Persisting & rehydrating data

Previous versions of zfy used to rely on a custom persistence solution. Fortunately, zustand added built-in support for this feature in [v3.1.4](https://github.com/pmndrs/zustand/releases/tag/v3.1.4). Therefore: zfy simply employs zustand's [persist middleware](https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data) and tries to offer an easier API to use.

{% hint style="info" %}
Refer to the [**`CreateStoreOptionsType.persist`**](../api/types/createstoreoptionstype.md#persist)to read the full API reference.
{% endhint %}

## Persistence

Data is persisted on a store to store basis. In order to make a store persist its `data`, you need to provide the `persist` option to [**`createStore()`**](../api/createstore.md) 3rd argument and specify which storage solution you'd like to use:

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage'
import { createStore } from '@colorfy-software/zfy'

import type { StoresDataType } from '../types'

export const initialState: StoresDataType['user'] = {
  id: '',
  likes: 0,
}

export default createStore<StoresDataType['user']>('user', initialState, {
  persist: { getStorage: () => AsyncStorage },
})
```

This means that you can have stores using different storage solutions depending on your use case (LocalStorage, Cookies, IndexedDB, etc).

{% hint style="info" %}
If you want to look into how to bundle stores by type of storage solutions used, for instance, refer to the [**Handling multiple stores**](handling-multiple-stores.md) guide.
{% endhint %}

{% hint style="danger" %}
<mark style="color:red;">You need to make sure that your storage solution provides a</mark><mark style="color:red;">`getItem()`</mark><mark style="color:red;">, a</mark><mark style="color:red;">`setItem()`</mark> <mark style="color:red;"></mark><mark style="color:red;">and a</mark><mark style="color:red;">`removeItem()`</mark><mark style="color:red;">(only needed for zustand v4).</mark>
{% endhint %}

If it does not, of course, you can always provide it yourself:

```typescript
import { MMKV } from 'react-native-mmkv'
import { createStore } from '@colorfy-software/zfy'

import type { StoresDataType } from '../types'

export const initialState: StoresDataType['user'] = {
  id: '',
  likes: 0,
}

export const storage = new MMKV({ id: 'user' })

export default createStore<StoresDataType['user']>('user', initialState, {
  persist: {
    getStorage: () => ({
      getItem: (name) => storage.getString(name) ?? null,
      setItem: (name, value) => storage.set(name, value),
      removeItem: (name) => storage.delete(name),
    }),
  },
})
```

From there you're all set. Just use `getState().update()` to update your store as covered in the [**Creating & using a store**](creating-and-using-a-store.md#update) guide and your `data` will be persisted.

## Rehydration

Similarly to the persistence solution, zfy directly uses zustand's rehydration under the hood and simply exposes it via a different (hopefully simpler) API.

Usually, when it comes to rehydration, you might want to hide all (or part) of your app while the necessary stores are rehydrating. zfy provides 2 solutions out of the box to do so: a component - [**`<PersistGate/>`**](../api/persistgate.md)  - and a React Hook, [**`useRehydrate()`**](../api/userehydrate.md).

### `<PersistGate />`

This component expects an array of `stores` and will take care of displaying its `children` only when all the stores will have been rehydrated. You can as well provide a `loader` if needed:

{% code title="src/App.tsx" %}
```typescript
import { SafeAreaView, Text } from 'react-native'
import { PersistGate } from '@colorfy-software/zfy'

import appStore from './stores/app-store'
import userStore from './stores/user-store'

const Loader = () => (
  <SafeAreaView>
    <Text>⏳ Loading...</Text>
  </SafeAreaView>
)

export default function App() {
  return (
    <PersistGate stores={[appStore, userStore]} loader={<Loader />}>
      <MyApp />
    </PersistGate>
  )
}
```
{% endcode %}

{% hint style="info" %}
Note that if you add stores in the`stores` array that do not have the`persist` middleware enabled,`<PersistGate/>`(as well as `useRehydrate())`will automatically skip them.
{% endhint %}

### `useRehydrate()`

If `<PersistGate />` doesn't fit your need, you can have finer control over what happens via [**`useRehydrate()`**](../api/userehydrate.md):

{% code title="src/App.tsx" %}
```typescript
import { useEffect } from 'react'
import { SafeAreaView, Text } from 'react-native'
import { useRehydrate } from '@colorfy-software/zfy'

import appStore from './stores/app-store'
import userStore from './stores/user-store'

const Loader = () => (
  <SafeAreaView>
    <Text>Loading...</Text>
  </SafeAreaView>
)

export default function App() {
  const isRehydrated = useRehydrate([appStore, userStore])

  useEffect(() => {
    if (isRehydrated) {
      // 📍 Trigger side effect upon rehydration?
    }
  }, [isRehydrated])

  return isRehydrated ? <Loader /> : <MyApp />
}
```
{% endcode %}

{% hint style="info" %}
If you want to rehydrate/check the rehydration status of a given store directly, please refer to the corresponding section of the zustand's documentation:
{% endhint %}

{% embed url="https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data#rehydrate" %}

