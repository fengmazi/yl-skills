# 核心架构原则

## 1. 配置驱动 UI

表单和表格不由模板硬编码，而是用配置数组声明。

### 表单：`FormItem[][]` 声明

```ts
// useFormConfig.ts 中定义
export const formConfig = ref<FormItem[][]>([
  [
    { label: '单号', prop: 'billNo', type: 'input', placeholder: '请输入' },
    { label: '日期', prop: 'date', type: 'datePicker', valueFormat: 'YYYY-MM-DD' },
  ],
  [
    { label: '供应商', prop: 'supplierId', type: 'select', options: supplierOptions },
  ],
])
```

模板侧只做渲染：

```vue
<Form :config="formConfig" :model="formData" />
```

### 表格：`Column[]` 声明

```ts
// useColumn.ts 中定义
export const columns: Column[] = [
  { type: 'checkbox', width: 50 },
  { field: 'billNo', title: '单号', width: 160 },
  { field: 'supplierName', title: '供应商', width: 200 },
  { field: 'amount', title: '金额', width: 120, align: 'right', type: 'money' },
  { title: '操作', width: 180, slots: { default: 'action' }, fixed: 'right' },
]
```

---

## 2. Hook 组合式开发

页面逻辑通过 hooks 编排，setup 中按固定顺序组合：

```
useTable → useFormConfig → useCurd → useDetail / useOperate
```

每个 hook 职责单一：

| Hook | 输入 | 输出 |
|------|------|------|
| `useTable` | 查询 API + 查询条件 | `tableData`, `loading`, `pagination`, `onSearch` |
| `useFormConfig` | 枚举/档案数据 | `formConfig` 响应式配置 |
| `useCurd` | CRUD API + 刷新回调 | `handleAdd`, `handleEdit`, `handleDelete` |
| `useDetail` | 详情 API | `detailVisible`, `detailData`, `openDetail` |
| `useOperate` | 审批 API | `handleApprove`, `handleReject`, `handleRevoke` |

页面 setup 典型结构：

```vue
<script setup lang="ts">
// 1. 表格
const { tableData, loading, pagination, onSearch } = useTable(url, queryParams)

// 2. 枚举/档案
const { enums } = useEnum('groupNo')
const { docMap } = useDoc('docType')

// 3. 表单配置
const { formConfig } = useFormConfig(enums, docMap)

// 4. 列配置
const columns = useColumn(enums)

// 5. CRUD
const { handleAdd, handleEdit, handleDelete } = useCurd(url, onSearch)

// 6. 详情
const { detailVisible, detailData, openDetail } = useDetail()
</script>
```

---

## 3. JSX / TSX 渲染

项目中有三种 TSX 使用模式，按场景选择：

### 模式对比

| 模式 | 文件类型 | 定义方式 | 有无模板 | 有无 scoped 样式 | 适用场景 |
|------|---------|---------|---------|-----------------|---------|
| A. 纯渲染组件 | `.vue` | `defineComponent` + render 函数 | 无 | 有 | 通用 UI 组件，需样式隔离 |
| B. 页面组件 | `.vue` | `<script setup>` + `<template>` | 有 | 有 | 页面/业务组件，TSX 仅用于动态配置 |
| C. Hook 内组件 | `.tsx` | `function Component()` 在 hook 内 | 无 | 无 | 弹窗组件，与 hook 状态紧密耦合 |

---

### 模式 A：`.vue` + `defineComponent` + render 函数

纯渲染组件，无 `<template>`，全部用 TSX render 函数输出。用于 `components/common/` 下的通用 UI 组件。

```vue
<script lang="tsx">
import { defineComponent } from 'vue'
import type { PropType } from 'vue'

export default defineComponent({
  props: {
    list: {
      required: true,
      type: Array as PropType<any[]>,
    },
  },
  setup(props) {
    return () => (
      <ul class="attchments">
        {props.list.length > 0
          ? props.list.map((item: any) => (
              <li onClick={() => handleClick(item)}>{item.name}</li>
            ))
          : '无数据'}
      </ul>
    )
  },
})
</script>

<style lang="scss" scoped>
.attchments { /* ... */ }
</style>
```

> 代表文件：`Attchments.vue`、`Timeline.vue`、`Detail.vue`、`DataTable.vue` 等 39 个文件。

---

### 模式 B：`.vue` + `<script lang="tsx" setup>` + `<template>`

页面级/业务组件的主流写法。setup 中用 TSX 编写复杂配置（列配置、表单配置），模板保持简洁。

```vue
<script lang="tsx" setup>
import { ref } from 'vue'

const columns = ref<Column[]>([
  { type: 'checkbox', width: 50 },
  {
    field: 'categoryName',
    title: '分类名称',
    minWidth: 200,
    filterParam: { type: String },
    treeNode: true,
  },
  {
    field: 'status',
    title: '状态',
    formatter: ({ cellValue }) => <el-tag>{statusMap[cellValue]}</el-tag>,
  },
])
</script>

<template>
  <Container>
    <DataTable :columns="columns" :data="tableData" />
  </Container>
</template>

<style lang="scss" scoped>
.page-container { /* ... */ }
</style>
```

> 代表文件：`docMaterialCategory.vue`、大部分 `views/` 下的业务页面。

---

### 模式 C：`.tsx` 纯文件 + hook 内定义组件函数

组件函数定义在 hook 闭包内，直接访问 hook 响应式状态，无需 props 传递。用于审批弹窗、详情弹窗等与业务逻辑紧密耦合的场景。

```tsx
// useOperate.tsx
import { ref, reactive } from 'vue'

export default function useOperate(refreshTable?: Function) {
  const dialogVisible = ref(false)
  const formData = reactive<OptFormData>({})

  const handleConfirm = () => {
    http.post(url, formData).then(() => {
      dialogVisible.value = false
      refreshTable?.()
    })
  }

  // 组件函数：直接访问闭包中的 dialogVisible、formData
  function OptDialog(props: DialogProps) {
    return (
      <el-dialog v-model={dialogVisible.value} title="审批" {...props.dialogAttr}>
        {{
          default: () => <Form formConfig={formConfig.value} formData={formData} />,
          footer: () => (
            <span>
              <el-button onClick={() => (dialogVisible.value = false)}>取消</el-button>
              <el-button type="primary" onClick={handleConfirm}>确定</el-button>
            </span>
          ),
        }}
      </el-dialog>
    )
  }

  return { handleOpt, OptDialog, OptPageDialog }
}
```

页面中使用：

```vue
<script setup lang="tsx">
const { handleOpt, OptDialog } = useOperate(onSearch)
</script>

<template>
  <div>
    <el-button @click="handleOpt({ opt: '审批', rowId: row.id, url: '...', param: {...} })">
      审批
    </el-button>
    <OptDialog />
  </div>
</template>
```

> 代表文件：`hooks/useOperate.tsx`、`hooks/useCurd.tsx`、`hooks/useDetail.tsx`、`hooks-nnw/useOperate.tsx` 等 13 个文件。

---

### 选择决策

```
需要 scoped 样式？
├─ 是 → 用 .vue 文件
│      ├─ 纯渲染、无模板逻辑 → 模式 A (defineComponent + render)
│      └─ 有模板结构、TSX 只做动态配置 → 模式 B (script setup + template)
└─ 否 → 组件与 hook 状态耦合？
       ├─ 是 → 模式 C (.tsx hook 内定义)
       └─ 否 → 模式 B（最通用）
```

---

## 4. 枚举双轨制

| 类型 | 存放位置 | 使用方式 | 适用场景 |
|------|---------|---------|---------|
| 静态枚举 | `src/enums/` 目录下 `*.ts` 文件 | 直接 import enums | 前端固定选项（性别、是否、管控类型等） |
| 动态枚举 | 服务端维护 | `useEnum(groupNo)` | 业务枚举（单据状态、审批结果） |

### 静态枚举

在 `src/enums/` 下新建 `xxx.ts`，export default 一个 key-value 对象：

```ts
// src/enums/controlType.ts
export default {
  FC: '厂控',
  SC: '自控',
}
```

`src/enums/index.ts` 会自动 glob 所有 `.ts` 文件，生成两个导出：

| 导出 | 结构 | 用途 |
|------|------|------|
| `enums.xxxMap` | `{ FC: '厂控', SC: '自控' }` | 值→标签映射，用于 formatter 或显示转换 |
| `enums.xxxOption` | `[{ value: 'FC', label: '厂控' }, ...]` | 用于 select/radio 的 options |

**关键用法**：代码值转中文标签用 `enums.xxxMap[value] || value`，安全回退到原值：

```ts
// 表单 text 字段显示转中文
data.controlType = enums.controlTypeMap[data.controlType] || data.controlType

// 表格列 formatter
formatter: ({ cellValue }) => enums.controlTypeMap[cellValue] || cellValue
```

### 动态枚举

```ts
const { enums } = useEnum('PO_STATUS')
// enums.value → [{ label: '待审核', value: '0' }, { label: '已审核', value: '1' }]
```

---

## 5. 迁移隔离（双系统共存）

迁入代码放 `-xxx` 后缀目录，全局注册先 xxx 后 common 保证覆盖。详见 `references/dual-system.md`。
