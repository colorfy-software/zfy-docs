# StoreType

{% hint style="info" %}
Interface representing the data structure of any [zfy store](../../guides/creating-and-using-a-store.md)'s `getState()`output.
{% endhint %}

{% code title="src/types.ts" %}
```typescript
import type { State } from 'zustand'

interface StoreType<StoreDataType> extends State {
  name: string
  data: StoreDataType
  reset: () => void
  update: (producer: (data: StoreDataType) => void) => void
}
```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L16-L21" %}

## API reference

### `name`

```typescript
  name: string
```

Name of the store to create.

{% hint style="danger" %}
<mark style="color:red;">Must be unique per store (</mark>[<mark style="color:orange;">ref</mark>](https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data#name)<mark style="color:red;">).</mark>
{% endhint %}

### `data`

```typescript
  data: StoreDataType
```

Data you want to save in your store. Can be of any type. As the zustand doc states:&#x20;

> _You can put anything in it: primitives, objects, functions_.

### `reset`

```typescript
reset: () => void
```

Function that, once invoked, resets the store's `data` to the value of the `initialData` provided to [**`createStore()`**](../createstore.md)'s 2nd argument.

### `update`

```typescript
update: (producer: (data: StoreDataType) => void) => void
```

Function that allows you to update your `data`.  The current value is provided to the `producer` function and thanks to Immer: you don't have to care about immutability at all. Example:

```typescript
const dealershipStore = createStore('dealership', { flagship: 'Ford Mustang' })

dealershipStpre.getState().update(data => {
  data.flagship = 'Porsche Taycan'
})
```

{% hint style="success" %}
For convenience, we recommend extracting `update()` in a function once, and then using that function to perform changes. Example:

```typescript
const dealershipStore = createStore('dealership', {
  isOpened: true,
  flagship: 'Ford Mustang',
})
const updateDealership = dealershipStore.getState().update

updateDealership(data => {
  data.isOpened = false
})

updateDealership(data => {
  data.isOpened = true
  data.flagship = 'Porsche Taycan'
})
```
{% endhint %}

{% hint style="warning" %}
Be careful to respect Immer's rules when it comes to producing a new state:
{% endhint %}

{% embed url="https://immerjs.github.io/immer/return" %}
