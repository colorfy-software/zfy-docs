# Creating & using a store

zfy general philosophy is to always behave as you'd expect zustand to, unless when we can provide an even better experience. Creating a store is an example of such exceptions.

{% hint style="info" %}
If you're looking for how to bundle and manage several stores at once, look into the [**Handling multiple stores**](handling-multiple-stores.md) guide.
{% endhint %}

## Creation

[**`createStore()`**](../api-reference/createstore.md) is the function that allows you to create a zustand store. As your primary interface with the library, we tried to make it as simple to use as possible. It only expects the store **`name`**, its default **`data`** and you can also provide some **`options`**. That last argument is where you can enable the middlewares zfy provides out-of-the-box, like [**`persist`**](../api-reference/types/createstoreoptionstype.md#persist) or [**`log`**](../api-reference/types/createstoreoptionstype.md#persist) or provide your own via [**`customMiddlewares`**](../api-reference/types/createstoreoptionstype.md#custommiddlewares).

{% tabs %}
{% tab title="user-store.ts" %}
{% code title="src/stores/user-store.ts" %}
```typescript
import AsyncStorage from '@react-native-async-storage/async-storage'
import { createStore } from '@colorfy-software/zfy'

import type { StoresDataType } from '../types'

export const initialState: StoresDataType['user'] = {
  id: '',
  likes: 0,
}

export default createStore<StoresDataType['user']>('user', initialState, {
  log: true,
  persist: { getStorage: () => AsyncStorage },
})
```
{% endcode %}
{% endtab %}

{% tab title="types.ts" %}
{% code title="src/types.ts" %}
```typescript
export interface StoresDataType {
  user: {
    id: string,
    likes: number,
  }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Usage

We decided to follow a specific set of rules that dictate how we use zustand stores. These decisions trickled down into some aspects of zfy's API that apply here.

It's important to note that, of course, **zustand still works as you'd expect**, (meaning all the items imported `from 'zustand'`). zfy only aims at enhancing not replacing them.

{% hint style="danger" %}
<mark style="color:red;">Disregarding the points covered below will prevent zfy from showcasing its full potential (and will potentially lead to undesired behaviours).</mark>
{% endhint %}

### **How to access data**

The data you want to use and display in your app is explicitly put inside the `getState().data` returned by [**`createaStore()`**](../api-reference/createstore.md) and only there.

{% hint style="warning" %}
No matter which type of data you want to put inside a store: it will always be available from `getState().data`, not the top level `getState()`.
{% endhint %}

This might sound quite restrictive or overdoing it at first but such an approach helped us simplify and harmonize how stores are created and used throughout the codebase. This is also what powers some of zfy's core features.

That's why:

{% hint style="success" %}
Any store created with zfy always exposes the same 4 elements from the `getState()` object: **`name`**,**`data`**,**`update()`**&**`reset()`**. And we expect you to only need & use those 4 elements.
{% endhint %}

Therefore, accessing your data will always look the same. Eg:

{% tabs %}
{% tab title="React" %}
```typescript
import shallow from 'zustand/shallow'

import userStore from './stores/user-store'
import settingsStore from './stores/settings-store'

const Component = (): JSX.Element => {
  const appLanguage = settingsStore(data => data.language)
  const [firstName, lastName] = userStore(
    (data) => [data.firstName, data.lastName],
    shallow
  )
  
  // ...
}
```
{% endtab %}

{% tab title="Vanilla JS" %}
```typescript
import settingsStore from './stores/settings-store'

const appLanguage = settingsStore.getState().data.appLanguage
```
{% endtab %}
{% endtabs %}

### **How to update data**

zfy conveniently exposes 2 methods for updating a store:  [`update()`](creating-and-using-a-store.md#update) & [`reset()`](creating-and-using-a-store.md#reset).

#### **`update()`**&#x20;

This is the method you'll be using the most as that's how `data` is being modified. Thanks to our use of Immer, you won't have to think about actions, reducers or even immutability. Just update your data, [even with mutable update patterns](https://immerjs.github.io/immer/update-patterns/), Immer will take care of the rest. Example:

{% tabs %}
{% tab title="React" %}
{% code title="src/screens/Login.tsx" %}
```typescript
import core from '../core'
import navigation from '../navigation'
import appInfoStore from '../stores/app-info-store'

const updateAppInfo = appInfoStore.getState().update

const Login = (): JSX.Element => {
 const onPressLogin = async (email, password) => {
  try {
    // ...
    await core.user.login(email, password)

    // 👇
    updateAppInfo(data => {
      data.lastLoginAt = Date.now()
    })

    await core.messages.fetchInbox()

    navigation.to('Home')
  } catch (error) {
    // handle error
  } finally {
    // ...
  }
 }

 // ...
}

export default Login
```
{% endcode %}
{% endtab %}

{% tab title="Vanilla JS" %}
{% code title="src/core.js" %}
```typescript
import Api from '../api'
import userStore from '../stores/user-store'
import messagesStore from '../stores/messages-store'

const updateUser = userStore.getState().update
const updateMessages = messagesStore.getState().update

export default {
  user: { 
    login: async (email, password) => new Promise((resolve, reject) => {
      try { 
        const userData = await Auth.login(email, password)
        
        // 👇
        updateUser(data => {
          data = userData
        })

        resolve(userData)
      } catch (error) {
        reject(error)
      }
    })
  },

  messages: {
    fetchInbox: async () => new Promise((resolve, reject) => {
      try {
        if (!(await Auth.isLoggedIn())) {
          return resolve(messagesStore.getState().data.inbox)
        }

        const inboxMessages = await Api.fetchInbox()

        // 👇
        updateMessages(data => {
          data.inbox = inboxMessages
        })

        resolve(inboxMessages)
      } catch (error) {
        reject(error)
      }
    }),

    markAsRead: async (messageId) => new Promise((resolve, reject) => {
      try { 
        if (!(await Auth.isLoggedIn())) return resolve(false)

        await Api.markMessageAsRead(messageId)
        
        // 👇
        updateMessages(data => {
          const index = data.inbox.findIndex(item => item.id === messageId)
        
          if (index !== -1) {
            messages.data.read.unshift({
              ...messages.data.inbox[index],
              readAt: date.now(),
            })
            messages.data.inbox.splice(index, 1)
          }
        })

        resolve(true)
      } catch (error) {
        reject(error)
      }
    })
  }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you've enabled the provided [**persist**](../api-reference/types/createstoreoptionstype.md#persist) middleware on a store, **`update()` ** will automatically take care of saving `data` in a way that will allow rehydration to work without you having to do anything else.
{% endhint %}

#### **`reset()`**&#x20;

This is your go-to method when you simply want to reset your store to its initial default data. It can be useful for when your users are logging out for instance:

{% tabs %}
{% tab title="Profile.tsx" %}
{% code title="src/screens/Profile.tsx" %}
```typescript
import core from '../core'
import navigation from '../navigation'

const Profile = (): JSX.Element => {
 const onPressLogout = async () => {
  try {
    // ...
    await core.user.logout()
    navigation.reset('Login')
  } catch (error) {
    // handle error
  } finally {
    // ...
  }
 }

 // ...
}

export default Profile
```
{% endcode %}
{% endtab %}

{% tab title="core.js" %}
{% code title="src/core.js" %}
```typescript
import Auth from 'my-auth-provider'

import userStore from '../stores/user-store'

const resetUser = userStore.getState().reset

export default {
  user: { 
    logout: async (email, password) => new Promise((resolve, reject) => {
      try { 
        await Auth.logout()
        // 👇
        resetUser()
        resolve(true)
      } catch (error) {
        reject(error)
      }
    })
  },
}hi
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you're wondering how to do all of this but with multiple stores at once, look into the [**Handling multiple stores**](handling-multiple-stores.md) guide.
{% endhint %}
