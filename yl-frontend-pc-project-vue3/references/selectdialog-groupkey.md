# SelectDialog 的 groupKey 特性

当需要在选择弹窗中按某个字段自动分组勾选时（如勾选某行后自动选中同入库单的所有行），在 SelectDialog 上设置 `groupKey`：

```vue
<SelectDialog
  v-model="showDialog"
  url="/api/queryDetail"
  rowId="rowKey"
  :columns="columns"
  :multiple="true"
  groupKey="orderId"
  @change="handleSelected"
/>
```

## 效果

- 勾选某行时，所有 `groupKey` 值相同的行自动选中
- 勾选不同 `groupKey` 值的行时，之前选中的行自动取消

## 实现原理

SelectDialog 内部 `checkboxChange` 事件中根据 `groupKey` 字段值批量操作复选框。
