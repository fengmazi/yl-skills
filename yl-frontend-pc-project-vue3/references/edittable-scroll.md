# EditTable 横向滚动条跳回问题

## 现象

EditTable 子表列过多出现横向滚动条时，在靠后的列输入会导致滚动条自动跳回开头。

## 原因

`reloadData()` 会完全重建 grid body，将 `scrollLeft` 复位为 0。

## 解决方案

deep watcher 中**不要调用 `reloadData()`**：

```ts
// ❌ 错误：会导致 scrollLeft 复位
watch(
  () => tableData,
  () => {
    emit('update:modelValue', tableData.value)
    xTable.value && xTable.value.reloadData(tableData.value)
  },
  { deep: true }
)

// ✅ 正确：注释掉 reloadData 并加说明，保留代码备后续参考
watch(
  () => tableData,
  () => {
    emit('update:modelValue', tableData.value)
    // 注意：reloadData 会导致横向滚动条跳回开头，不要启用
    // xTable.value && xTable.value.reloadData(tableData.value)
  },
  { deep: true }
)
```

旧版 `common-nnw/EditTable.vue` 早已注释掉此行，可作为参考。
