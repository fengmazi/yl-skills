# SelectDialog / DialogTable 翻页多选勾选丢失

## 现象

弹窗中选择核算项目（分页列表），在：
- 第1页勾选了若干条
- 翻到第2页勾选
- 翻回第1页 → **前页勾选的记录全部消失了**

全选后翻页也会自动取消。

## 原因

`DataTable.vue` 底层 vxe-grid 配置了 `checkbox-config={{ reserve: true }}`，vxe-table **原生支持跨页保留勾选**。

但 `SelectDialog.vue` 和 `DialogTable.vue` 在每次 `tableData` 变化（翻页、筛选等）时都执行 `setAllCheckboxRow(false)`，主动清除了 vxe-table 保留的勾选记录，破坏了 reserve 机制。

```ts
// ❌ 问题代码：每次翻页都清空勾选
const resetSelect = () => {
  nextTick(() => {
    selectTable.value?.$refs.xTable.setAllCheckboxRow(false)  // 清除所有页的勾选
    // 再从 selectedData 恢复...
  })
}

watch(tableData, resetSelect)  // 翻页即触发
```

**完整时序：**

```
弹窗打开 → tableData 加载第1页 → resetSelect → setAllCheckboxRow(false) → 恢复 selectedData 勾选
用户勾选第1页几条
翻第2页   → tableData 加载第2页 → resetSelect → setAllCheckboxRow(false) ← 清掉了第1页的 reserve!
翻回第1页 → tableData 加载第1页 → resetSelect → setAllCheckboxRow(false) ← 清掉了第2页的 reserve!
```

## 解决方案

只在弹窗打开后首次加载时清空并初始化勾选，后续翻页不再强制清空，由 vxe-table 的 `reserve: true` 自然保留跨页勾选。

### SelectDialog.vue 修改

```diff
+let needResetCheckbox = true
+
+watch(() => props.dialogData.visible, (visible) => {
+  if (visible) {
+    needResetCheckbox = true
+  }
+})

 const resetSelect = () => {
   nextTick(() => {
     const { selectedData, rowId } = props.dialogData
-    selectTable.value?.$refs.xTable.setAllCheckboxRow(false)
+    if (needResetCheckbox) {
+      needResetCheckbox = false
+      selectTable.value?.$refs.xTable.clearCheckboxReserve()
+      selectTable.value?.$refs.xTable.setAllCheckboxRow(false)
+    }
     if (selectedData) {
       const loop = (list: any) => {
         list.map((item: any) => {
           if (selectedData.find((v: any) => v[rowId] === item[rowId])) {
             selectTable.value?.$refs.xTable.setCheckboxRow(item, true)
           }
           if (item.children) {
             loop(item.children)
           }
         })
       }
       loop(tableData.value)
     }
   })
 }
```

### DialogTable.vue 修改

同样思路，在 `queryPage` 的回调中加标志位判断，监听 `modelValue` 变化时重置标志位。

## 关键 API

| API | 说明 |
|-----|------|
| `setAllCheckboxRow(false)` | 清空当前页所有勾选，同时清除 reserve 中的当前页行 |
| `clearCheckboxReserve()` | 清除所有跨页保留的勾选记录 |
| `getCheckboxRecords(true)` | 获取当前页勾选的行 |
| `getCheckboxReserveRecords(true)` | 获取所有跨页保留的勾选行（其他页的） |
| `setCheckboxRow(item, true)` | 勾选指定行，自动加入 reserve |

## 影响分析

| 场景 | 是否受影响 | 说明 |
|------|:--:|------|
| `isTree=true` 弹窗 | 否 | 无分页，`tableData` 一次性返回全部数据 |
| `selectedData` 始终为空 | 否 | `resetSelect` 无恢复逻辑 |
| `multiple=false` 单选模式 | 否 | 无需跨页保留 |
| `isTree=false` 且 `selectedData` 非空 | **是** | 约 11 个实例会受影响，全部修复 |

## 确认勾选收集逻辑

修复后确认保存的勾选收集方式不变，始终同时取当前页 + 跨页的勾选：

```ts
// saveData / checkboxChange 中：
const result = [
  ...selectTable.value?.$refs.xTable.getCheckboxRecords(true),       // 当前页
  ...selectTable.value?.$refs.xTable.getCheckboxReserveRecords(true), // 其他页
]
```
