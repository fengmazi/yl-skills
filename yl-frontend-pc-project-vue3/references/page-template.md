# 页面开发模板

## Vue 文件书写顺序

所有 `.vue` 文件必须按以下顺序组织：

```
<script> → <template> → <style>
```

**Why:** script 先定义组件的所有逻辑（props、data、computed、methods），template 再消费，style 最后做样式增强。阅读时先理解组件行为，再看结构和样式，层次分明。

```vue
<script lang="tsx" setup>
// 1. 所有逻辑在此
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <!-- 2. 模板渲染 -->
  <div>{{ count }}</div>
</template>

<style lang="scss" scoped>
/* 3. 样式 */
</style>
```

> 不得使用 `<template>` 在前的写法。

## 页面级组件目录规范

页面级组件（`src/views/` 下的路由页面）必须使用文件夹包裹 `index.vue`，禁止直接放置 `.vue` 文件：

```
# 正确
src/views/archive/docMaterialCategory/
└── index.vue

# 错误
src/views/archive/docMaterialCategory.vue
```

**Why:** 页面后续通常需要拆分 hooks、types、子组件等，提前预留文件夹避免日后迁移。即使页面当前只有一个文件，也用文件夹 + `index.vue`。

子组件、hooks、types 均在文件夹内就近存放：

```
src/views/archive/docMaterialCategory/
├── index.vue          # 页面主文件
├── useTable.ts        # 表格数据逻辑
├── useFormConfig.ts   # 搜索表单配置
├── useColumn.ts       # 表格列配置
├── useCurd.ts         # CRUD 操作
└── types.ts           # 类型定义
```

---

## 列表页标准模板

新增业务页面时按以下结构搭建：

```
src/views/{module}/
├── index.vue          # 页面模板
├── useTable.ts        # 表格数据
├── useFormConfig.ts   # 搜索表单配置
├── useColumn.ts       # 表格列配置
├── useCurd.ts         # CRUD 操作（可选）
└── types.ts           # 类型定义
```

### index.vue

```vue
<script setup lang="ts">
import { useTable } from './useTable'
import { useFormConfig } from './useFormConfig'
import { useColumn } from './useColumn'
import { useCurd } from './useCurd'

const { tableData, loading, pagination, onSearch, onPageChange } = useTable()
const { formConfig, queryParams } = useFormConfig()
const { columns } = useColumn()
const { curdState, handleEdit, handleDelete, handleCurdSubmit } = useCurd(onSearch)
</script>

<template>
  <div class="page-container">
    <!-- 搜索区域 -->
    <Form :config="formConfig" :model="queryParams" @search="onSearch" />

    <!-- 表格区域 -->
    <DataTable
      :columns="columns"
      :data="tableData"
      :loading="loading"
      :pagination="pagination"
      @page-change="onPageChange"
    >
      <template #action="{ row }">
        <el-button link type="primary" @click="handleEdit(row)">修改</el-button>
        <el-button link type="danger" @click="handleDelete(row)">删除</el-button>
      </template>
    </DataTable>

    <!-- CRUD 弹窗 -->
    <CurdDialog
      v-model="curdState.visible"
      :mode="curdState.mode"
      :config="formConfig"
      :data="curdState.data"
      @submit="handleCurdSubmit"
    />
  </div>
</template>
```

### useTable.ts

```ts
import { ref } from 'vue'
import { getList } from '@/api/module'

export function useTable() {
  const queryParams = ref({ billNo: '', date: '' })

  const {
    tableData,
    loading,
    pagination,
    onSearch,
    onPageChange,
  } = useTable('/api/module/list', queryParams)

  return { tableData, loading, pagination, queryParams, onSearch, onPageChange }
}
```

### useFormConfig.ts

```ts
import { computed } from 'vue'
import type { FormItem } from '@/types'

export function useFormConfig() {
  const queryParams = ref({ billNo: '', date: '' })

  const formConfig = computed<FormItem[][]>(() => [
    [
      { label: '单号', prop: 'billNo', type: 'input' },
      { label: '日期', prop: 'date', type: 'datePicker', valueFormat: 'YYYY-MM-DD' },
    ],
  ])

  return { formConfig, queryParams }
}
```

### useColumn.ts

```ts
import type { Column } from '@/types'

export function useColumn() {
  const columns: Column[] = [
    { type: 'checkbox', width: 50 },
    { field: 'billNo', title: '单号', width: 160 },
    { field: 'amount', title: '金额', width: 120, align: 'right', type: 'money' },
    { title: '操作', width: 180, slots: { default: 'action' }, fixed: 'right' },
  ]

  return { columns }
}
```

### useCurd.ts

```ts
export function useCurd(onRefresh: () => void) {
  return useCurd('/api/module', onRefresh)
}
```

---

## 审批页面附加层

审批类页面在列表页基础上额外引入：

```ts
// 详情
const { detailVisible, detailData, openDetail } = useDetail()
// 审批操作
const { handleApprove, handleReject, handleRevoke } = useOperate('/api/module', onSearch)
```

操作列增加审批按钮：

```vue
<template #action="{ row }">
  <el-button link type="primary" @click="openDetail(row)">详情</el-button>
  <el-button v-if="row.status === 'PENDING'" link type="success" @click="handleApprove(row)">
    审核
  </el-button>
</template>
```

---

## 批量操作模式

实现批量送审、批量审批等批量操作的标准模式。核心思想：**状态驱动工具栏 + checkboxConfig 控制可选行 + filterParam 自动筛选数据**。

### 1. 状态类型与初始值

```ts
type BtnState = 'default' | 'submit' | 'check'
const btnState = ref<BtnState>('default')
```

### 2. checkboxConfig（控制勾选逻辑）

**必须用 ref 包裹**，通过 `checkboxConfig` 透传属性覆盖 DataTable 内部 vxe-grid 的 `checkbox-config`，不能用 DataTable 的 `checkMethod` prop（内部静态绑定，动态切换不生效）：

```ts
const checkboxConfig = ref({
  checkMethod: ({ row }: { row: any }): boolean => false,
})
```

模板中传 `checkboxConfig`（非 `checkMethod`）作为透传属性：

```html
<DataTable
  ref="dataTableRef"
  :columns="columns"
  :checkboxConfig="checkboxConfig"
  ...
/>
```

### 3. toolbarBtns（状态驱动工具栏）

```tsx
const toolbarBtns = () => {
  switch (btnState.value) {
    case 'submit':
      return [
        <el-button type="primary" size="small" onClick={handleBatchSubmit}>
          确认批量送审
        </el-button>,
        <el-button type="info" size="small" onClick={cancelBatch}>
          取消
        </el-button>,
      ]
    case 'check':
      return [
        <el-button type="primary" size="small" onClick={() => batchCheck('pass')}>
          批量通过
        </el-button>,
        <el-button type="danger" size="small" onClick={() => batchCheck('reject')}>
          批量驳回
        </el-button>,
        <el-button type="info" size="small" onClick={cancelBatch}>
          取消
        </el-button>,
      ]
    default:
      return [
        <el-radio-group ...>...</el-radio-group>,
        <el-button ... onClick={() => startBatch('submit')}>批量送审</el-button>,
        <el-button ... onClick={() => startBatch('check')}>批量审批</el-button>,
      ]
  }
}
```

### 4. startBatch / cancelBatch（进入/退出批量模式）

修改 `statusColumns`（审批状态列）的 `filterParam` 来触发 DataTable 的响应式筛选链路。DataTable 内部通过 `newColunms` computed → `filterOptions` → `querys` computed → `watchEffect` 自动 emit `tableRefresh`，**不要手动调 `handleTableRefresh()`**：

```ts
const startBatch = (state: BtnState) => {
  btnState.value = state
  dataTableRef.value?.clearCheckbox()
  if (state === 'submit') {
    checkboxConfig.value.checkMethod = ({ row }: { row: any }) =>
      ['new'].includes(row.approveStatus)
    statusColumns.value[0].filterParam = {
      type: Array,
      options: enums.approveStatusOption,
      operator: 'EQUAL',
      value: 'new',
    }
  } else if (state === 'check') {
    checkboxConfig.value.checkMethod = ({ row }: { row: any }) =>
      ['submit', 'abandon'].includes(row.approveStatus)
    statusColumns.value[0].filterParam = {
      type: Array,
      options: enums.approveStatusOption,
      operator: 'IN',
      value: ['submit', 'abandon'],
    }
  }
}

const cancelBatch = () => {
  btnState.value = 'default'
  statusColumns.value[0].filterParam = {
    type: Array,
    options: enums.approveStatusOption,
  }
  checkboxConfig.value.checkMethod = ({ row }: { row: any }) => false
  dataTableRef.value?.clearCheckbox()
}
```

### 5. 批量操作执行函数

```ts
// 批量送审
const handleBatchSubmit = () => {
  if (!tableSelected.length) return ElMessage.warning('请至少选择一条记录')
  ElMessageBox.confirm('确定要批量送审吗？', '提示')
    .then(() =>
      http.post('/xxx/submit', tableSelected.map((item: any) => item.orderId))
        .then((res: any) => {
          ElMessage.success(res.message)
          dataTableRef.value?.clearCheckbox()
          tableSelected.length = 0
          cancelBatch()
        })
    )
    .catch(() => {})
}

// 批量审批（通过/驳回共用，通过 status 参数区分）
const batchCheck = (status: 'pass' | 'reject') => {
  if (!tableSelected.length) return ElMessage.warning('请至少选择一条记录')
  http.post('/xxx/check', {
    ids: tableSelected.map((item: any) => item.orderId),
    approveStatus: status,
  }).then((res: any) => {
    ElMessage.success(res.message)
    dataTableRef.value?.clearCheckbox()
    tableSelected.length = 0
    cancelBatch()
  })
}
```

### 6. 勾选事件

```ts
let tableSelected: any[] = []
const checkboxChange = ({ records }: { records: any }) => {
  tableSelected = records
}
```

### 关键点

| 要点 | 说明 |
|------|------|
| **透传 checkboxConfig** | DataTable 的 `checkMethod` prop 是静态绑定，必须用 `:checkboxConfig="checkboxConfig"` 透传到 vxe-grid |
| **filterParam 驱动筛选** | 不要直接改 `queryList`，修改列上的 `filterParam.value`+`filterParam.operator`，让 DataTable 内部响应式链路自动触发 |
| **不要手动 handleTableRefresh** | filterParam 变化 → DataTable 自动 emit `tableRefresh`，手动调用会重复请求 |
| **操作完清勾选并退模式** | `clearCheckbox()` + `tableSelected.length = 0` + `cancelBatch()` |
| **filterParam 重置** | cancelBatch 时重置为 `{ type: Array, options: ... }`（无 value 不筛选） |

---

## 文件命名约定

| 文件 | 导出方式 | 命名 |
|------|---------|------|
| `useTable.ts` | `export function` | `use{Module}Table`（如重名冲突时） |
| `useFormConfig.ts` | `export function` | `use{Module}FormConfig` |
| `useColumn.ts` | `export function` | `use{Module}Column` |
| `useCurd.ts` | `export function` | `use{Module}Curd` |
| `types.ts` | `export interface` | PascalCase |

---

## filterParam 筛选配置类型

表格列的 `filterParam` 支持四种筛选类型：

| type | 渲染控件 | 示例 |
|------|---------|------|
| `String` | 文本输入 | `filterParam: { type: String }` |
| `Number` | 数字输入 | `filterParam: { type: Number }` |
| `Date` | 日期选择 | `filterParam: { type: Date }` |
| `Array` | 下拉选择 | `filterParam: { type: Array, options: enums.statusOption }` |

### 常用枚举选项

- `enums.statusOption` — 启用/停用
- `enums.booleanOption` — 是/否
- `enums.approveStatusOption` — 审批状态
- `enums.orderStatusOption` — 订单状态

### 动态选项

```ts
const column = getColumn('fieldName')
column.filterParam!.options = res.options
```

> **重要**：如果通过 `getColumn` 动态赋值 `options`，该列的 `filterParam.type` 必须是 `Array`，不能是 `String`（`String` 渲染为文本输入框，`Array` 渲染为下拉选择）。
>
> ```ts
> // 正确：定义为 Array 类型，options 初始为 []
> { field: 'purchaseTypeName', title: '采购类型', filterParam: { type: Array, options: [] } }
> // 后续通过 getColumn 注入选项
> getColumn('purchaseTypeName').filterParam!.options = res.options
> ```

### 单据/明细列表切换

页面上有"单据""明细"两个列表切换时（如入库单/出库单），注意以下几点：

**1. tableUrl 使用 computed 自动切换**

单据和明细调不同的接口。**不要**在 `handleListTypeChange` 里手动赋 `tableUrl.value`，用 `computed` 让 URL 随 `listType` 自动计算。避免 DataTable 的 `watchEffect` 和 `useTable` 的 `watch(url)` 同时触发导致重复请求：

```ts
// 正确：computed 自动切换
const listType = ref('order')
const tableUrl = computed(() =>
  listType.value === 'order' ? '/logisticsInbound/query' : '/logisticsInbound/queryDetail'
)

const handleListTypeChange = (val: string) => {
  listType.value = val
  dataTableRef.value?.clearCheckbox()
}
```

```ts
// 错误：手动赋值 URL，会导致 useTable 的 watch(url) 和 DataTable 双触发
const handleListTypeChange = () => {
  tableUrl.value = listType.value === 'order' ? '/xxx/query' : '/xxx/queryDetail'
}
```

**2. 不要加 `:key="listType"`**

加 `:key` 会让 DataTable 完全卸载重建，其内部的 `watchEffect` 会在挂载时触发一次请求，加之 `useTable` 的 `watch(url)` 也触发一次，导致列表接口请求两次。

```html
<!-- 正确：不加 :key，columns/rowId 通过 reactive props 自动更新 -->
<DataTable
  ref="dataTableRef"
  :rowId="listType === 'order' ? 'orderId' : 'detailId'"
  :columns="listType === 'order' ? columns : detailColumns"
  ...
/>
```

**3. JSX 中 radio-group 用 modelValue + onUpdate:modelValue**

`v-model={ref.value}` 在 JSX 中退化为单向绑定，切换后 `listType.value` 不会更新。`v-model={ref}` 在某些 JSX 插件版本下也会报 "Assignment to constant variable"。用显式的 `modelValue` + `onUpdate:modelValue` 规避：

```tsx
const toolbarBtns = () => [
  <el-radio-group
    size="small"
    modelValue={listType.value}
    onUpdate:modelValue={handleListTypeChange}
    style="margin-right: 10px"
  >
    <el-radio-button value="order">单据</el-radio-button>
    <el-radio-button value="detail">明细</el-radio-button>
  </el-radio-group>,
]
```

**4. rowId 必须动态切换**

单据模式用单据级别的唯一 ID（如 `orderId`），明细模式用明细级别的唯一 ID（如 `detailId`），否则 vxe-table 会因主键重复报错。

**5. 列定义分开维护**

两种模式下的表格列通常不同（单据列 vs 明细列），通过 `listType` 切换：

```ts
const listType = ref('order')

const { columns } = useColumn([
  // ... 单据列定义
])

const detailColumns = ref<Column[]>([
  // ... 明细列定义
])
```

---

## 常用列配置

### 操作人 / 操作时间

列表页展示操作人和操作时间时，字段名和配置如下：

```ts
{ field: 'optUserName', title: '操作人', filterParam: { type: String } },
{
  field: 'optDate',
  title: '操作时间',
  width: 215,
  filterParam: { type: Date, attrs: { valueFormat: 'x' } },
  formatter: ({ cellValue }) => formatTime(cellValue, 'yyyy年MM月dd日 hh:mm'),
},
```

- `optUserName` — 操作人名称（后端统一字段）
- `optDate` — 操作时间，int64 时间戳，列宽优先用 **215**

---

## 报表页面模式

报表页面分为两种：**固定列报表**（有分页）和**动态列报表**（无分页，后端返回 `head` 定义表头）。两种都放在 `src/views/report/` 目录下，路由在 `src/router/module/report.ts`。

### 固定列报表（带分页 + 合计行）

适用场景：列结构固定、后端返回 `{ records, total }` 分页数据的报表。参照 `materialSummary.vue`、`supplierSummary.vue`。

**核心要素**：
- `Container` + `DataTable` 组件
- `useTable`（分页）+ `useColumn`（列定义）
- `queryParams`（reactive）同步筛选条件
- 月份选择器 + 月末修正 watch
- 合计行 `footerMethod`
- 导出按钮调用 `downloadFile`

```vue
<template>
  <Container>
    <DataTable
      :columns="columns"
      :tableData="tableData"
      :total="total"
      :currentPage="pageNo"
      :pageSize="pageSize"
      :loading="loading"
      :toolbarBtns="toolbarBtns"
      :show-footer="true"
      :footer-method="footerMethod"
      @tableRefresh="handleQueryPage"
    />
  </Container>
</template>

<script setup lang="tsx">
import { ref, reactive, watch, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import useTable from '@/hooks/useTable'
import useColumn from '@/hooks/useColumn'
import { downloadFile, formatMoney } from '@/utils'

const startTime = ref(new Date(new Date().getFullYear(), new Date().getMonth(), 1).getTime())
const endTime = ref(new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0, 23, 59, 59, 999).getTime())

// month picker 返回当月第一天，结束月份需修正为当月最后一天
watch(endTime, (val) => {
  const d = new Date(val)
  if (d.getDate() === 1) {
    endTime.value = new Date(d.getFullYear(), d.getMonth() + 1, 0, 23, 59, 59, 999).getTime()
  }
})

const queryParams = reactive<Record<string, any>>({})

// 列定义：金额类字段用 formatMoney formatter + summary: true 开启合计
const { columns } = useColumn([
  { type: 'seq', title: '序号', width: 60 },
  { field: 'field1', title: '列1', minWidth: 200 },
  { field: 'money', title: '金额', width: 150, attrs: { precision: 2 }, summary: true,
    formatter: ({ cellValue }: any) => formatMoney(cellValue) },
])

const { tableData, total, pageNo, pageSize, loading, handleQueryPage: originalHandleQueryPage, sum } = useTable({
  url: ref('/report/xxx'),
  params: queryParams,
})

// 筛选条件同步
watch([startTime, endTime], () => {
  queryParams.startTime = startTime.value
  queryParams.endTime = endTime.value
}, { immediate: true })

onMounted(() => {
  handleQueryPage(1, pageSize.value, [], [])
})

// 合计行：有 seq 列时"合计"放 seq 列位置，否则放 index 0
const footerMethod = () => {
  const s = sum.value as any
  if (!s || Object.keys(s).length === 0) return []
  return [columns.value.map(col => {
    if (col.field === 'money') return formatMoney(s.money)
    if (col.type === 'seq') return '合计'
    return ''
  })]
}

const validateQuery = () => {
  if (!startTime.value || !endTime.value) {
    ElMessage.warning('请选择月份区间')
    return false
  }
  if (startTime.value > endTime.value) {
    ElMessage.warning('开始月份不能大于结束月份')
    return false
  }
  return true
}

const handleQueryPage = (currentPage: number, size: number, querys: QueryDO[], sorts: SortDO[]) => {
  if (!validateQuery()) return
  originalHandleQueryPage(currentPage, size, querys, sorts)
}

// JSX toolbar：月份选择器 + 查询 + 导出
const toolbarBtns = () => [
  <div style="display:flex;align-items:center;gap:8px;flex-wrap:wrap;">
    <el-date-picker v-model={startTime.value} type="month" size="small" style="width:140px;"
      value-format="x" placeholder="开始月份" />
    <span>至</span>
    <el-date-picker v-model={endTime.value} type="month" size="small" style="width:140px;"
      value-format="x" placeholder="结束月份" />
    <el-button type="primary" size="small"
      onClick={() => { if (validateQuery()) handleQueryPage(1, pageSize.value, [], []) }}>查询</el-button>
    <el-button type="warning" size="small"
      onClick={() => downloadFile('/report/exportXxx', '报表名.xlsx', 'post', {
        startTime: startTime.value,
        endTime: endTime.value,
      })}>导出</el-button>
  </div>,
]
</script>
```

### 动态多级表头报表（不分页）

适用场景：列结构由后端 `head` 数组动态决定、返回全量数据（不分页）的报表。参照 `deptMaterialOutSummary.vue`、`progressOverview.vue`。

**核心要素**：
- `usePage: false`（返回全量数组）
- 后端 `head` 数组递归构建 vxe-table 多级表头（`children`）
- `sum` 对象递归构建合计行
- 刷新按钮绑定自定义 `loadData`（需重建表头 + 更新数据）

```vue
<script setup lang="tsx">
import { ref, reactive } from 'vue'

const params = reactive({ startTime: startTime.value, endTime: endTime.value })

const {
  tableData, loading, handleQueryPage, sum: sumData,
} = useTable({
  url: ref('/report/deptMaterialOutSummary'),
  params,
  usePage: false,
})

const columns = ref<any[]>([])
const dynamicHead = ref<any[]>([])

// 加载数据：请求 + 重建表头 + 赋值
const loadData = () => {
  params.startTime = startTime.value
  params.endTime = endTime.value
  handleQueryPage(1, 9999, [], []).then((res: any) => {
    dynamicHead.value = res.head || []
    buildColumns(res.head || [])
    tableData.value = res.data || []
  })
}

// 递归构建多级列
const buildColumns = (head: any[]) => {
  const walkTree = (items: any[]): any[] => {
    return items.map((item: any) => {
      if (item.child && item.child.length > 0) {
        return { title: item.name, align: 'center', children: walkTree(item.child) }
      }
      return {
        field: item.id,
        title: item.name,
        width: item.id === 'deptCode' ? 160 : 100,
        align: 'center',
        formatter: item.id !== 'deptCode' ? ({ cellValue }: any) => {
          if (cellValue === null || cellValue === undefined) return ''
          const num = Number(cellValue)
          return isNaN(num) ? cellValue : num.toFixed(2)
        } : undefined,
      }
    })
  }
  columns.value = walkTree(head)
}

// 合计行：递归匹配 sum 对象的动态字段
const footerMethod = () => {
  const sum = sumData.value as any
  if (!sum || Object.keys(sum).length === 0) return []

  const buildFooterRow = (items: any[]): any[] => {
    return items.map((item: any) => {
      if (item.child && item.child.length > 0) return buildFooterRow(item.child)
      if (item.id === 'deptCode') return '合计'
      const val = sum[item.id]
      return val != null ? formatMoney(Number(val)) : ''
    })
  }

  return [buildFooterRow(dynamicHead.value).flat(Infinity) as any[]]
}

const toolbarBtns = () => [
  <div style="display:flex;align-items:center;gap:8px;flex-wrap:wrap;">
    <el-date-picker v-model={startTime.value} type="month" size="small" style="width:140px;"
      value-format="x" placeholder="开始月份" />
    <span>至</span>
    <el-date-picker v-model={endTime.value} type="month" size="small" style="width:140px;"
      value-format="x" placeholder="结束月份" />
    <el-button type="primary" size="small" onClick={() => { if (validateQuery()) loadData() }}>查询</el-button>
    <el-button type="warning" size="small"
      onClick={() => downloadFile('/report/exportXxx', '报表名.xlsx', 'post', {
        startTime: startTime.value, endTime: endTime.value,
      })}>导出</el-button>
  </div>,
]

loadData()
</script>
```

**模板注意**：
```html
<!-- 不分页报表必须绑定 @tableRefresh="loadData"，否则刷新按钮无效 -->
<DataTable
  rowId="deptCode"
  :columns="columns"
  :tableData="tableData"
  :loading="loading"
  :toolbarBtns="toolbarBtns"
  :show-footer="true"
  :footer-method="footerMethod"
  @tableRefresh="loadData"
/>
```

**关键差异**：
| 项目 | 固定列报表 | 动态列报表 |
|------|----------|-----------|
| `usePage` | `true`（默认） | `false` |
| 数据来源 | `res.data.records` | `res.data`（全量数组） |
| 列定义 | `useColumn([...])` 静态声明 | `buildColumns(head)` 动态构建 |
| 合计行 | 按固定字段取 `sum.xxx` | 递归遍历 `head` 取 `sum[item.id]` |
| 刷新事件 | `@tableRefresh="handleQueryPage"` | `@tableRefresh="loadData"`（需重建表头） |
