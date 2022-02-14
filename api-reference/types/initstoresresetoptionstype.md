# InitStoresResetOptionsType

{% hint style="info" %}
Interface of the `options`argument accepted by the [**`initStores().stores.reset()`**](initstorestype.md#stores.reset) method.
{% endhint %}

{% code title="src/types.ts" %}
```typescript
type InitStoresResetOptionsType<StoreDataType> = {
  omit?: Array<keyof StoreDataType>
}
```
{% endcode %}

{% embed url="https://github.com/colorfy-software/zfy/blob/main/src/types.ts#L71-L73" %}
