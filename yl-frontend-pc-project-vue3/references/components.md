# 核心组件

## DataTable

核心表格组件，集成分页、筛选、排序、合计行。

```vue
<DataTable
  ref="xTable"
  :columns="columns"
  :data="tableData"
  :loading="loading"
  :pagination="pagination"
  :show-summary="true"
  @page-change="handlePageChange"
  @sort-change="handleSortChange"
>
  <template #action="{ row }">...</template>
</DataTable>
```

**常用 props**：`columns`, `data`, `loading`, `pagination`, `show-summary`, `height`

**常用 events**：`page-change`, `sort-change`, `checkbox-change`, `cell-click`

**注意**：项目可能存在 `DataTable.vue` 的双系统版本（`common/DataTable.vue` vs `common-nnw/DataTable.vue`），事件名和筛选配置可能不同，参考既有页面写法。

### 树形表格展开/收起所有

当 DataTable 配置 `type="tree"` 时，可通过模板 ref 访问内部 vxe-grid 实例调用 `setAllTreeExpand`：

```vue
<script lang="tsx" setup>
import { useTemplateRef } from 'vue'

const xTable = useTemplateRef('xTable')

const toolbarBtns = () => [
  // ... 其他按钮
  <el-button type="primary" size="small" onClick={() => xTable.value?.$refs.xTable.setAllTreeExpand(true)}>
    展开所有
  </el-button>,
  <el-button type="primary" size="small" onClick={() => xTable.value?.$refs.xTable.setAllTreeExpand(false)}>
    收起所有
  </el-button>,
]
</script>

<template>
  <DataTable
    ref="xTable"
    type="tree"
    :columns="columns"
    :tableData="tableData"
    ...
  />
</template>
```

**原理**：`ref="xTable"` 获取 DataTable 组件实例，其内部的 `$refs.xTable` 是 vxe-grid 组件实例，`setAllTreeExpand(true/false)` 是 vxe-grid 的原生方法。

**注意**：Vue 3.5+ 项目使用 `useTemplateRef('refName')` 而非 `ref()` 获取模板引用。Vue < 3.5 的项目继续使用 `ref()`。

**常见错误**：不要用 `document.querySelector('.vxe-grid-container')` 获取 DOM 元素来调用此方法 — DOM 元素上不存在 `setAllTreeExpand`，必须通过 Vue 组件实例访问。

---

## Form + FormItem

配置驱动的表单引擎，通过 `FormItem[][]` 声明即可渲染。

```vue
<Form :config="formConfig" :model="formData" :rules="rules" label-width="100px" />
```

`FormItem` 常用属性：

| 属性 | 类型 | 说明 |
|------|------|------|
| `label` | `string` | 标签文本 |
| `prop` | `string` | 字段名，对应 `model` 的 key |
| `type` | `string` | 控件类型：`input`/`select`/`datePicker`/`number`/`textarea` |
| `placeholder` | `string` | 占位文本 |
| `options` | `Option[]` | select 的选项 |
| `valueFormat` | `string` | 日期格式化，如 `'YYYY-MM-DD'` |
| `disabled` | `boolean` | 是否禁用 |
| `span` | `number` | 栅格宽度（24 栅格） |

二维数组：第一层为行，第二层为该行的表单项。

```ts
const config: FormItem[][] = [
  // 第一行 2 个
  [{ label: '单号', prop: 'billNo', type: 'input' }, { label: '日期', prop: 'date', type: 'datePicker' }],
  // 第二行 1 个
  [{ label: '备注', prop: 'remark', type: 'textarea', span: 24 }],
]
```

---

## DialogForm

表单弹窗，封装了弹窗显隐、表单校验、关闭重置。

```vue
<DialogForm
  v-model="visible"
  :config="formConfig"
  :data="formData"
  :rules="rules"
  title="新增单据"
  @submit="handleSubmit"
/>
```

**关键行为**：关闭时自动 `resetFields()`，`submit` 前自动 `validate()`。

---

## CurdDialog

CRUD 弹窗，配合 `useCurd` 使用，内部渲染 DialogForm。

```vue
<CurdDialog
  v-model="curdState.visible"
  :mode="curdState.mode"
  :config="formConfig"
  :data="curdState.data"
  @submit="handleCurdSubmit"
/>
```

`mode` 为 `'add'`/`'edit'`，控制标题和初始行为。

---

## EditTable

可编辑子表格，常用于单据的明细行编辑。

```vue
<EditTable
  ref="editTableRef"
  v-model="detailData"
  :columns="editColumns"
  :rules="editRules"
/>
```

**关键坑**：deep watcher 中不要调用 `reloadData()`，否则横向滚动条跳回开头。详见 `references/edittable-scroll.md`。

---

## PageDialog

全页遮罩弹窗，用于需要大面积展示的内容（如完整的详情页、导入预览）。

```vue
<PageDialog v-model="visible" title="导入预览" width="90%">
  <DataTable :columns="columns" :data="previewData" />
</PageDialog>
```

---

## DetailDialog

审批详情弹窗，用于展示审批流程、操作日志等。

```vue
<DetailDialog
  v-model="visible"
  :bill-id="currentBillId"
  :bill-type="billType"
/>
```

---

## SelectDialog

数据选择弹窗，支持多选、搜索、分组勾选。

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

**`groupKey` 特性**：按字段自动分组勾选。详见 `references/selectdialog-groupkey.md`。

---

## OptDialog

审批/撤销/弃审等操作的对话框，由 `useOperate` 渲染。无需手动配置，`useOperate` 自动处理。

```ts
const { handleOpt, OptDialog } = useOperate(handleTableRefresh)

// 打开操作弹窗
handleOpt({
  opt: '审批',       // 操作类型
  rowId: row.id,
  url: '/api/check',
  param: { businessId: row.id, taskTypeCode: '1001001' }
})
```

```vue
<OptDialog :dialogAttr="{ width: '800px' }" />
```

**操作类型与表单内容**：

| 类型 | 表单内容 |
|-----|---------|
| 审批 | 审批状态（通过/驳回）、审批意见、附件 |
| 撤销 | 撤销原因 |
| 弃审 | 弃审原因 |
| 红冲 | 红冲类型、附件 |

---

## UploadFile

文件上传组件，用于表单中的附件字段。通常通过表单配置的 `type: 'upload'` 自动渲染，也可直接使用：

```vue
<UploadFile v-model={formData.accList} allowExt={['pdf', 'doc', 'docx', 'xls', 'xlsx']} />
```

---

## Timeline

时间轴组件，用于展示审批流程历史：

```ts
import Timeline from '@/components/common-bt/Timeline.vue'
// 查看流程时间轴
handleShowHistory(id, type)
```

---

## PieChart / BarChart

```ts
import PieChart from '@/components/common-bt/PieChart.vue'
import BarChart from '@/components/common-bt/BarChart.vue'
```

图表组件基于 ECharts 封装，用于仪表盘和统计页面。
