# 审批流可编辑明细表

审批流中允许特定审批节点修改明细数据（编辑字段、新增行、从外部数据源选取）的完整实现模式。

## 适用场景

- 申请部门填写的明细有误或遗漏，需要后续审批节点（如投资管理科）修正/补充
- 审批流中某一步需要审批人从价格库/物料库等外部数据源选取并追加明细行
- 审批流中某一步需要审批人修改明细的特定字段（数量、金额、备注等），并自动重算关联字段

## 涉及文件

| 文件 | 作用 |
|------|------|
| `components/common-bt/Detail.vue` | 审批详情渲染，切换为可编辑表格 |
| `components/common/Task.vue` | 任务收件箱，审批提交时构建 `data.data` 载荷 |
| `enums-bt/taskTypeCode.ts` | 任务类型码常量 |

---

## 实现步骤

### 1. Detail.vue：引入 EditTable 和 SelectDialog

```ts
import EditTable from '@/components/common-bt/EditTable.vue'
import SelectDialog from '@/components/common-bt/SelectDialog.vue'
```

### 2. Detail.vue：定义 SelectDialog 的列和状态

```ts
const showPriceSelect = ref(false)
const priceColumns = ref<Column[]>([
  { field: 'priceCode', title: '结算项目编码', width: 150, filterParam: { type: String } },
  { field: 'priceName', title: '结算项目名称', filterParam: { type: String } },
  { field: 'specModel', title: '规格型号', width: 200, showOverflow: true },
  { field: 'unitMeasureName', title: '计量单位', width: 100 },
  {
    field: 'money',
    title: '结算单价',
    width: 120,
    filterParam: { type: Number },
    formatter: ({ cellValue }) => (cellValue ? Number(cellValue).toFixed(2) : ''),
  },
])
```

### 3. Detail.vue：将 DataTable 替换为 EditTable

原来用 `DataTable` 只读展示明细，审批节点改为 `EditTable` 可编辑：

```tsx
// 原来（只读）
<DataTable
  style="width: 100%;height:300px;"
  rowId="detailId"
  loading={false}
  pager-config={{ enabled: false }}
  toolbar-config={{ enabled: false }}
  columns={checkDetailColumns}
  tableData={detail.value.detailList}
  show-footer={true}
  footer-method={() => calcFooter(checkDetailColumns, detail.value.detailList)}
/>

// 改为
<EditTable
  style="width: 100%;"
  v-model={detail.value.detailList}
  colunms={checkDetailColumns}
  edit={['editCheckPrice'].includes(props.taskStep)}
  copy={false}
  buttons={['editCheckPrice'].includes(props.taskStep) ? [
    <el-button size="small" type="primary" onClick={() => handleAddCheckDetail()}>
      新增行
    </el-button>,
  ] : []}
/>
```

**关键点**：
- `edit` prop 控制是否可编辑，仅在指定 `taskStep` 时为 `true`
- `buttons` 仅在编辑步骤显示，放「新增行」等操作按钮
- `v-model` 双向绑定 `detail.value.detailList`，编辑结果直接反映在数据上

### 4. Detail.vue：列配置 — 条件可编辑 + 自动计算

```ts
const checkDetailColumns: Column[] = [
  // 只读列：editRender.enabled = false
  { field: 'priceCode', title: '结算项目编码', width: 150, editRender: { enabled: false } },
  { field: 'priceName', title: '结算项目名称', minWidth: 200, editRender: { enabled: false } },
  { field: 'specModel', title: '规格型号', width: 150, showOverflow: true, editRender: { enabled: false } },
  { field: 'unitMeasureName', title: '计量单位', width: 100, editRender: { enabled: false } },
  { field: 'perMoney', title: '结算单价', width: 120, editRender: { enabled: false }, formatter: ... },

  // 可编辑列：仅在指定 taskStep 时启用编辑
  {
    field: 'amount',
    title: '数量',
    width: 120,
    editRender: { enabled: ['editCheckPrice'].includes(props.taskStep), name: 'default' },
    slots: ['editCheckPrice'].includes(props.taskStep)
      ? {
          edit: ({ row }: { row: any }) => {
            return [
              <el-input-number
                v-model={row.amount}
                placeholder="数量"
                precision={2}
                min={0}
                controls-position="right"
                size="small"
                style="width:100%;"
                onChange={() => {
                  const amount = Number(row.amount) || 0
                  const perMoney = Number(row.perMoney) || 0
                  row.money = Number((amount * perMoney).toFixed(2))
                }}
              />,
            ]
          },
        }
      : {
          default: ({ row }: { row: any }) => [row.amount ? Number(row.amount).toFixed(2) : ''],
        },
  },

  // 备注列：el-input 编辑
  {
    field: 'remark',
    title: '备注',
    width: 200,
    editRender: { enabled: ['editCheckPrice'].includes(props.taskStep), name: 'default' },
    slots: ['editCheckPrice'].includes(props.taskStep)
      ? {
          edit: ({ row }: { row: any }) => {
            return [<el-input v-model={row.remark} placeholder="请输入备注" size="small" clearable />]
          },
        }
      : {
          default: ({ row }: { row: any }) => [row.remark || ''],
        },
  },

  // 金额列：由数量和单价自动计算，不可编辑
  {
    field: 'money',
    title: '金额',
    width: 120,
    summary: true,
    attrs: { precision: 2 },
    editRender: { enabled: false },
    formatter: ({ row }: { row: any }) => {
      const amount = Number(row.amount) || 0
      const perMoney = Number(row.perMoney) || 0
      row.money = Number((amount * perMoney).toFixed(2))
      return row.money.toFixed(2)
    },
  },
]
```

**关键点**：
- `editRender: { enabled: false }` → 始终只读
- `editRender: { enabled: condition }` → 条件可编辑
- `slots.edit` 用 JSX 返回自定义编辑器（`el-input-number`、`el-input` 等）
- `slots.default` 定义非编辑状态下的只读渲染
- 关联字段（如 `money`）在 `formatter` 中重算确保展示值准确

### 5. Detail.vue：新增行处理

```ts
const handleAddCheckDetail = () => {
  showPriceSelect.value = true
}

const handlePriceSelectConfirm = (records: any[]) => {
  if (!records || !records.length) {
    ElMessage.warning('请至少选择一条记录')
    return
  }
  if (!detail.value.detailList) {
    detail.value.detailList = []
  }
  records.forEach((price: any) => {
    detail.value.detailList.push({
      // ⚠️ 必须带上当前单据的外键 ID，否则提交时后端匹配不上
      checkId: detail.value.checkId,
      priceId: price.priceId,
      priceCode: price.priceCode,
      priceName: price.priceName,
      specModel: price.specModel,
      unitMeasureName: price.unitMeasureName,
      perMoney: price.money,  // SelectDialog 中字段名可能是 money
      amount: 1,              // 默认数量
      money: price.money * 1, // 初始金额 = 单价 × 默认数量
      remark: '',
    })
  })
  showPriceSelect.value = false
}
```

**⚠️ 常见 bug**：新增行务必带上当前单据的外键 ID（如此处的 `checkId`），否则后端审批提交时找不到对应记录。这就是 `b327b6f` 补充的修复。

### 6. Detail.vue：SelectDialog 挂载

```tsx
<SelectDialog
  v-model={showPriceSelect.value}
  rowId="priceId"
  multiple={true}
  url="/priceInfo/query"
  columns={priceColumns.value}
  selected={[]}
  querys={[
    { property: 'priceStatus', value: 'U', operator: 'EQUAL' },
    // 可选：按当前单据的业务分类过滤
    ...(detail.value?.categoryId
      ? [{ property: 'categoryId', value: [detail.value.categoryId], operator: 'IN' }]
      : []),
  ]}
  onChange={handlePriceSelectConfirm}
/>
```

### 7. Detail.vue：向 Task.vue 暴露 detail 数据

Task.vue 提交审批时需要读取编辑后的明细数据，因此 Detail.vue 需在 `detail` 变化时 emit：

```ts
watch(
  () => detail.value,
  newVal => {
    detailStore.setDetailData(detail.value)
    if (props.taskTypeCode === '1001035') {
      emit('detail', detail.value)
    }
  },
  { deep: true, immediate: true }
)
```

**注意**：父组件 Task.vue 需监听 `@detail` 事件接收数据（已有机制，无需额外开发）。

### 8. Task.vue：审批提交时构建 data 载荷

在 `handleTaskConfirm()` 中，针对可编辑审批步骤构建 `data.data`：

```ts
// 小型项目验收审批 — 提交编辑后的明细数据
if (currentTask.taskTypeCode === '1001035' && ['editCheckPrice'].includes(currentTaskStep.value)) {
  if (!taskDetail.value?.detailList || taskDetail.value.detailList.length === 0) {
    return ElMessage.warning('请至少添加一条明细数据')
  }
  data.data = {
    detailList: taskDetail.value.detailList.map((item: any) => ({
      checkId: item.checkId,
      priceId: item.priceId,
      amount: item.amount,
      money: item.money,
      remark: item.remark,
    })),
  }
}
```

**关键点**：
- 校验明细不为空
- 只提交后端需要的字段（不要全量透传 `detailList`）
- 确保 `checkId` 含在每条明细中（对应新增行的修复）

---

## 完整 checklist

实现一个审批流可编辑明细表时，逐项确认：

- [ ] `Detail.vue` 引入 `EditTable`、`SelectDialog`
- [ ] 定义 SelectDialog 列配置（`priceColumns`）
- [ ] 原 `DataTable` 替换为 `EditTable`，`edit` 和 `buttons` 按 `taskStep` 条件控制
- [ ] 各列配置 `editRender.enabled` 按需设为 `false` 或条件表达式
- [ ] 可编辑列配置 `slots.edit` JSX 渲染编辑器
- [ ] 关联计算字段（如 `money = amount * perMoney`）在 `onChange` 或 `formatter` 中重算
- [ ] 新增行处理函数 `handlePriceSelectConfirm` 带上外键 ID
- [ ] 新增行默认值合理（如 `amount: 1`）
- [ ] SelectDialog 查询条件匹配当前业务上下文
- [ ] `detail` watch 中 emit 数据给 Task.vue
- [ ] Task.vue `handleTaskConfirm` 中校验 + 构建 `data.data`
- [ ] 跑 `pnpm run typecheck` 通过

---

## 相关组件文档

- EditTable：见 `references/components.md`
- SelectDialog：见 `references/components.md` 和 `references/pagination-checkbox-reserve.md`
- el-input-number 性能问题：见 `references/el-input-number-performance.md`
