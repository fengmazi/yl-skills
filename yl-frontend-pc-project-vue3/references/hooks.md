# 核心 Hooks

> **目录命名约定**：无后缀目录（`hooks/`、`components/common/`、`enums/`、`utils/`）是项目**原本的**代码。带后缀的（`hooks-nnw/`、`hooks-bt/`、`components/common-nnw/`、`enums-nnw/`、`utils-nnw/` 等）是从其他项目**迁入**的代码，因与原项目耦合度高连带迁入，后缀标识来源（`nnw`=南泥湾、`bt`=宝塔等，具体后缀因项目而异）。详见 SKILL.md 中"迁移后缀说明"。

## useTable

分页表格数据管理。封装了请求、分页、排序、筛选状态。

```ts
interface UseTableConfig {
  url: Ref<string>           // API 路径（响应式）
  method?: 'POST' | 'GET'    // 请求方式，默认 POST
  usePage?: boolean          // 是否分页，默认 true
  querys?: QueryDO[]         // 初始筛选条件
  size?: number              // 默认每页条数，默认 20
  params?: Reactive<Record<string, any>>   // 额外请求参数（筛选条件等）
}

const {
  tableData,        // Ref<T[]>            表格数据
  total,            // Ref<number>         总条数
  pageNo,           // Ref<number>         当前页码
  pageSize,         // Ref<number>         每页条数
  loading,          // Ref<boolean>        加载状态
  sum,              // Ref<Record<string, any>>  合计行数据
  queryList,        // Ref<QueryDO[]>      当前筛选条件
  sortList,         // Ref<SortDO[]>       当前排序
  handleTableRefresh,     // () => Promise   刷新（保持当前状态）
  handleQueryPage,        // (currentPage, size, querys, sorts) => Promise  分页/筛选/排序触发
  buildFooterMethod,      // (columns) => VxeTablePropTypes.FooterMethod    生成合计行方法
} = useTable(config)
```

**请求体格式**：`useTable` 自动将请求拼装为 `{ querys, sorts, current, pageSize, ...params }`。当 `usePage: false` 时不传分页参数，返回 `res.data` 直接作为 `tableData`（不分页数组）。

**典型用法（分页列表）**：

```ts
// 基础用法：只传 url，DataTable 点击分页自动调用 handleQueryPage
const { tableData, total, pageNo, pageSize, loading, handleQueryPage } = useTable({
  url: ref('/checkAcceptOrder/queryApplication'),
})
```

**典型用法（带筛选参数）**：

```ts
const queryParams = reactive<Record<string, any>>({})

const { tableData, total, pageNo, pageSize, loading, handleQueryPage: originalHandleQueryPage, sum } = useTable({
  url: ref('/report/supplierSummary'),
  params: queryParams,
})

// 筛选条件变化时同步到 queryParams
watch([startTime, endTime], () => {
  queryParams.startTime = startTime.value
  queryParams.endTime = endTime.value
}, { immediate: true })

// 包装一层校验
const handleQueryPage = (currentPage: number, size: number, querys: QueryDO[], sorts: SortDO[]) => {
  if (!validateQuery()) return
  originalHandleQueryPage(currentPage, size, querys, sorts)
}
```

**模板绑定**：

```html
<DataTable
  :columns="columns"
  :tableData="tableData"
  :total="total"
  :currentPage="pageNo"
  :pageSize="pageSize"
  :loading="loading"
  @tableRefresh="handleQueryPage"
/>
```

**分页参数说明**：`useTable` 内部用 `current` + `pageSize` 传给后端，与 DataTable 的 `currentPage` / `pageSize` props 配合。不要在 `params` 里手动加 `pageNo` 或 `pageSize`。

**不分页用法**（如动态报表返回全量数据）：

```ts
const { tableData, loading, handleQueryPage, sum } = useTable({
  url: ref('/report/deptMaterialOutSummary'),
  params,
  usePage: false,
})

// usePage: false 时，handleQueryPage 不会传 current/pageSize
// 返回的 res.data 直接赋值给 tableData（数组）
```

---

## useCurd

新增/修改/删除操作管理。

```ts
const {
  curdState,       // Reactive<{ visible, mode, data }>
  handleAdd,       // () => void                    打开新增弹窗
  handleEdit,      // (row: T) => void              打开编辑弹窗
  handleDelete,    // (row: T) => Promise<void>     删除确认
  handleSubmit,    // (data: T) => Promise<void>    提交表单
} = useCurd('/api/entity', onSearch)
```

**参数**：
- `baseUrl: string` — CRUD API 基础路径
- `onRefresh: () => void` — 操作成功后的刷新回调
- `options?: { beforeSubmit?, afterSubmit? }` — 生命周期钩子

---

## useFormConfig

表单配置生成，依赖枚举和档案数据。

```ts
const { formConfig } = useFormConfig(enums, docMap)
```

返回的 `formConfig` 自动根据枚举/档案生成 select 选项。

---

## useColumn

表格列配置。

```ts
const columns = useColumn(enums)
```

返回的 `columns` 中，枚举类型字段自动关联选项映射，用于 `formatter` 显示 label。

---

## useEnum

服务端动态枚举获取。

```ts
const { enums, loading } = useEnum('PURCHASE_STATUS')
// enums.value → [{ label: '待审核', value: 'PENDING' }, ...]
```

**参数**：`groupNo: string` — 枚举分组号

**缓存**：多次调用同一 `groupNo` 会复用缓存，不会重复请求。`useEnum` 内部有缓存层。

**路径差异**：不同项目的导入路径可能不同（`@/hooks-bt/useEnum` vs `@/hooks/useEnum`），以当前项目实际路径为准。

---

## 合计行（useTable 的 sum 与 footerMethod）

`useTable` 内置了默认的 `footerMethod`，但不推荐直接使用。**合计行必须在本页面覆盖重写**，不应依赖 useTable 的默认实现。

### 原因

- 默认 footerMethod 统一 `toFixed(2)`，无法适配不同精度需求（如金额需 4 位 + 千分位 `formatMoney`）
- 默认 "合计" 固定在第 0 列，不能跟随是否有选择列动态调整位置

### 规范

1. **合计优先在页面覆盖重写**：从 `useTable` 解构 `sum`，在页面内写 `footerMethod`，不要使用 useTable 默认导出的 `footerMethod`

2. **"合计"位置规则**：
   - 列表**有选择列（checkbox）** → "合计"放在第 **2** 列（index 1，即选择列后面一列）
   - 列表**无选择列**（全是展示列） → "合计"放在第 **1** 列（index 0）

3. **格式化与列定义保持一致**：footerMethod 中各字段的格式化方式应与该列 `formatter` 一致

### 示例

```tsx
// 从 useTable 解构 sum
const { tableData, total, pageNo, pageSize, loading, queryList, sum, handleTableQuery } =
  useTable('/materialOutbound/query')

// 有选择列的 detailColumns — "合计"在 index 1
const { columns: detailColumns } = useColumn([
  { type: 'checkbox', width: 50 },          // index 0
  { field: 'approveStatus', title: '审批状态' }, // index 1 ← "合计"放这里
  { field: 'amount', title: '数量', ... },
  { field: 'taxMoney', title: '金额', ... },
])

const footerMethod: VxeTablePropTypes.FooterMethod = ({ columns }: { columns: any[] }) => {
  return [
    columns.map((col: any, index: number) => {
      if (index === 1) return '合计'  // 有选择列，放第二列
      if (['amount'].includes(col.field) && sum.value[col.field] != null) {
        return Number(sum.value[col.field]).toFixed(4)
      }
      if (['taxMoney'].includes(col.field) && sum.value[col.field] != null) {
        return formatMoney(sum.value[col.field], 4)
      }
      return ''
    }),
  ]
}
```

```tsx
// 无选择列的 detailColumns — "合计"在 index 0
const { columns: detailColumns } = useColumn([
  { field: 'approveStatus', title: '审批状态' },  // index 0 ← "合计"放这里
  { field: 'amount', title: '数量', ... },
])

const footerMethod: VxeTablePropTypes.FooterMethod = ({ columns }: { columns: any[] }) => {
  return [
    columns.map((col: any, index: number) => {
      if (index === 0) return '合计'  // 无选择列，放第一列
      ...
    }),
  ]
}
```

### 模板配置

```html
<DataTable
  :show-footer="listType === 'detail'"
  :footer-method="listType === 'detail' ? footerMethod : undefined"
  ...
/>
```

---

## useDoc

档案数据获取（如部门、人员、费用项目等基础档案）。

```ts
const { docMap, docList, loading } = useDoc('SUPPLIER')
// docMap.value → { '1': '供应商A', '2': '供应商B' }
// docList.value → [{ id: '1', name: '供应商A' }, ...]
```

### 常用档案类型

| type 值 | 说明 | 备注 |
|--------|-----|------|
| dept | 部门（带权限） | 使用 `deptStatus` 字段判断停用 |
| dept-no-per | 部门（不带权限） | 使用 `status` 字段判断停用 |
| personal | 人员列表 | — |
| role | 角色列表 | — |
| group | 权限组 | — |
| user | 用户下拉 | — |
| feeItem | 费用项目 | — |
| relevantDept | 归口部门 | — |
| model | 模板列表 | — |

### 返回值结构

```ts
getDoc({ type: 'dept' }).then(res => {
  // res.options  — 下拉选项（含 disabled 状态）
  // res.data     — 原始数据
  // res.map      — id-name 映射
  // res.filterOptionsList — 扁平选项（用于表格筛选）
})
```

**部门停用差异**：`dept` 类型用 `deptStatus === 'S'` 判断停用，`dept-no-per` 类型用 `status === 'S'` 判断停用。如需停用部门在下拉中不可选，使用 `dept` 类型。

---

## useDetail

审批详情查看。

> `hooks-nnw/useDetail` 是从南泥湾项目迁移过来的版本（源项目：`costcontrol-admin-container`），`hooks/useDetail` 是本项目原有版本。API 有差异，见下。

### hooks/（项目原有）

```ts
const {
  handleShowDetail,    // (id, code, title?, onlyCustomFooter?) => void
  handleShowPrint,     // (id, code) => void  打印预览
  handleClose,         // () => void          关闭弹窗
  DetailDialog,        // 详情弹窗组件（el-dialog）
  DetailPageDialog,    // 详情弹窗组件（PageDialog）
} = useDetail()
```

**DetailDialog 和 DetailPageDialog 都支持默认插槽**，传入的内容渲染在弹窗 footer/header 区域。常用于添加自定义按钮（如整单复制、表头复制）：

```html
<DetailDialog :dialogAttr="{ width: '1200px' }">
  <el-button type="primary" @click="copyRecord('all')">整单复制</el-button>
  <el-button type="primary" @click="copyRecord('head')">表头复制</el-button>
</DetailDialog>
```

**`onlyCustomFooter` 参数**：第 4 参为 `true` 时隐藏默认的"确定"按钮，只显示插槽内容。用于完全自定义 footer。

### hooks-nnw/（从南泥湾迁入）

```ts
const {
  handleShowDetail,    // (id, code, title?, show?, width?) => void
  handleShowPrint,     // (id, code) => void
  handleDialogClose,   // () => void          关闭弹窗（含 activeName 重置）
  renderDetailDialog,  // 详情弹窗组件（el-dialog）
  DetailDialog,        // 详情弹窗组件（PageDialog）
} = useDetail()
```

**renderDetailDialog 支持默认插槽**，内容渲染在 footer 区域。`DetailDialog` 支持默认插槽，内容渲染在 header 区域。

**`show` 参数**：第 4 参控制 `showSlot`，为 `true` 时才渲染插槽内容（隐藏默认关闭按钮）。

**`width` 参数**：第 5 参设置弹窗宽度，默认 `'1200px'`。

### 常见模式：详情弹窗加复制按钮

详见 `references/copy-record.md`。

---

## useOperate

审批操作（审核、弃审、撤销、红冲）。

```ts
const {
  handleApprove,   // (row: T) => Promise<void>
  handleReject,    // (row: T, reason: string) => Promise<void>
  handleRevoke,    // (row: T) => Promise<void>
  handleOpt,       // (config) => void  通用操作入口
  OptDialog,       // 操作弹窗组件
} = useOperate('/api/purchase', onRefresh)
```

触发审批后自动弹窗确认、调用 API、刷新列表。

### handleOpt 通用操作入口

```ts
// 审批
handleOpt({
  opt: '审批',
  rowId: row.id,
  url: '/api/check',
  param: { businessId: row.id, taskTypeCode: '1001001' }
})

// 撤销
handleOpt({
  opt: '撤销',
  rowId: row.id,
  url: '/api/revocation',
  param: { businessId: row.id, taskTypeCode: '1001001' }
})

// 弃审
handleOpt({
  opt: '弃审',
  rowId: row.id,
  url: '/api/abandon',
  param: { businessId: row.id, taskTypeCode: '1001001' }
})
```

**操作类型**：`审批` / `撤销` / `弃审` / `红冲`。不同类型自动渲染对应的表单（审批意见、撤销原因等）。

### 常见审批状态约束

```ts
// 审批中可撤销
disabled={row.approveStatus !== 'submit' && row.approveStatus !== 'check'}

// 审批通过可弃审
disabled={row.approveStatus !== 'pass'}

// 新建状态可编辑/删除
disabled={['submit', 'check', 'pass'].includes(row.approveStatus)}
```

---

## useImport

Excel 导入管理。

```ts
const {
  importVisible,    // Ref<boolean>
  importUrl,        // string
  handleImport,     // () => void    打开导入弹窗
  handleUpload,     // (file) => void 处理上传
} = useImport('/api/purchase/import', onSearch)
```

---

## usePrint

单据打印。

```ts
const { handlePrint } = usePrint()
// handlePrint(billId, billType) — 打开打印预览
```
