# Getting started

[zustand](https://github.com/pmndrs/zustand) is an amazing library that keeps state management as simple as you would have always loved it to be. zfy has been built on top of it to provide a useful set of tools to make that experience even more delightful.&#x20;

## 🏗️ Installation

{% tabs %}
{% tab title="Yarn" %}
```bash
yarn add @colorfy-software/zfy
```
{% endtab %}

{% tab title="npm" %}
```bash
npm install @colorfy-software/zfy
```
{% endtab %}

{% tab title="Expo" %}
```
expo install @colorfy-software/zfy
```
{% endtab %}
{% endtabs %}

## 🌺 Features

* Fully typed with TypeScript
* Standardized access/update API backed by Immer
* Ability to manage & consume multiples stores at once
* Simple API for store creation with custom middlewares
* Out-of-the-box persist gate component & rehydration hook
* Logger, persist & subscribe middlewares available via a simple flag

## 💻 Usage

1. :hatching\_chick: Create your stores

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage'
import {
  PersistGate,
  createStore,
  initStores,
  useRehydrate,
} from '@colorfy-software/zfy'

const simpleStore = createStore('groceries', ['kimchi', 'bok choy', 'tteokbokki' ])
const persistedStore = createStore(
  'user',
  { firstName: 'Jolyne' },
  { persist: { getStorage: () => AsyncStorage } },
)
```

2\. :recycle: Rehydrate your data if needed

```typescript
import { SafeAreaView, Text } from 'react-native'

const Loader = () => (
  <SafeAreaView>
    <Text>⏳ Loading...</Text>
  </SafeAreaView>
)

const App = () => (
  <PersistGate stores={[persistedStore]} loader={<Loader />}>
    <MyApp />
  </PersistGate>
)

// or

const App = () => {
  const isRehydrated = useRehydrate([persistedStore])
  return isRehydrated ? <Loader /> : <MyApp />
}
```

3\. :package: Bundle your stores as you please

```typescript
const { stores, useStores } = initStores([simpleStore, persistedStore])
console.log(stores.simpleStore.getState().data[2]) // 'tteokbokki'

// ...or don't!

console.log(persistedStore.getState().data.firstName) // 'Jolyne'
```

4\. :atom: Use the bundled/individual stores in your components

```typescript
const Component = () => {
  const itemToPurchase = useStores('groceries', data => data[0]) // 'kimchi'
  const userName = useStores('user', data => data.firstName) // 'Joylyne'  
  return null
}

// or

const Component = () => {
  const itemToPurchase = simpleStore('groceries', data => data[0]) // 'kimchi'
  const userName = persistedStore('user', data => data.firstName) // 'Joylyne'
  return null
}
```

5\. :new: Update your data immutably (thanks to [Immer](https://immerjs.github.io/immer/)) from wherever

```typescript
stores.groceries.getState().update(data => {
  data[3] = 'chai'
})

// or

simpleStore.getState().update(data => {
  data[3] = 'chai'
})
```

6\. :wind\_blowing\_face: Reset it all once your user is done

```typescript
stores.reset()

// ...or one by one!

simpleStore.getState().reset()
persistedStore.getState().reset()
```

## 💫 Acknowledgments

As you can imagine: major thanks to [the team of contributors behind **zustand**](https://github.com/pmndrs/zustand/graphs/contributors) for such an amazing library! zfy also exists thanks to [the folks working on **Immer**](https://github.com/immerjs/immer/graphs/contributors) who made it so easy to deal with immutable state updates.
