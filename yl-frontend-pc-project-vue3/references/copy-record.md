# 整单复制 / 表头复制

在列表页的详情弹窗底部添加"整单复制"和"表头复制"按钮，点击后将原单据数据拷贝到新增表单中。

## 使用场景

- 单据内容相似，只需微调部分字段或多条明细时，复制后修改比从零新建快
- 适用于所有带 DetailDialog 的 CRUD 列表页（入库、出库、报销、调拨等）

## 两个模式

| 模式 | 行为 |
|------|------|
| **整单复制** (`'all'`) | 复制表单所有字段 + 明细列表（detailList），重置 ID 和日期 |
| **表头复制** (`'head'`) | 只复制表单字段，明细列表置空，用户手动添加明细 |

## 实现步骤

### 1. 模板：在 DetailDialog 内放按钮

> 说明：`renderDetailDialog` 是 hooks-nnw（从南泥湾迁入）的 useDetail 返回的；`DetailDialog` 是 hooks/（项目原有）的 useDetail 返回的。

**项目原有 hooks/ 版本**（`DetailDialog` 基于 `el-dialog`，按钮走 `footer` 插槽）：

```html
<DetailDialog :dialogAttr="{ width: '1200px' }">
  <el-button type="primary" @click="copyRecord('all')">整单复制</el-button>
  <el-button type="primary" @click="copyRecord('head')">表头复制</el-button>
</DetailDialog>
```

**hooks-nnw/（从南泥湾迁入）版本**（`renderDetailDialog` 基于 `el-dialog`，按钮走 `footer` 插槽）：

```html
<renderDetailDialog>
  <el-button type="primary" @click="copyRecord('all')">整单复制</el-button>
  <el-button type="primary" @click="copyRecord('head')">表头复制</el-button>
</renderDetailDialog>
```

### 2. 解构 useDetail 时取出 close 方法

```ts
// hooks/ 版本（项目原有）
const { handleShowDetail, handleClose, DetailDialog } = useDetail()

// hooks-nnw/ 版本（从南泥湾迁入）
const { handleShowDetail, handleDialogClose, renderDetailDialog } = useDetail()
```

### 3. 存储当前行 ID，包装 showDetail

```ts
const tempRowId = ref('')

const showDetail = (rowId: string, code: string) => {
  tempRowId.value = rowId
  handleShowDetail(rowId, code)
}
```

> 列表中点击单据号查看详情时，原来调 `handleShowDetail(row.orderId, '1001034')`，改为调 `showDetail(row.orderId, '1001034')`。

### 4. 编写 copyRecord 函数

```ts
const copyRecord = (type: 'all' | 'head') => {
  if (!tempRowId.value) return
  http.get(`/<baseUrl>/selectById/${tempRowId.value}`).then(res => {
    if (!res.data) return
    handleClose()  // 或 handleDialogClose()
    handleAdd('/<baseUrl>/add', beforeConfirm, true)
    Object.keys(formData).forEach(key => {
      ;(formData as any)[key] = (res.data as any)[key]
    })
    formData.orderId = undefined   // 主键 ID 字段置空
    formData.orderDate = Date.now() // 日期刷新为当天
    if (type === 'head') {
      formData.detailList = []
    }
  })
}
```

### 5. 更新列中的详情点击

列表中所有 `handleShowDetail(row.xxx, 'taskTypeCode')` 调用改为 `showDetail(row.xxx, 'taskTypeCode')`。

## 完整示例

以下以"材料入库单"页面为例，展示所有改动点：

```html
<!-- 模板：DetailDialog 嵌按钮 -->
<DetailDialog :dialogAttr="{ width: '1200px' }">
  <el-button type="primary" @click="copyRecord('all')">整单复制</el-button>
  <el-button type="primary" @click="copyRecord('head')">表头复制</el-button>
</DetailDialog>
```

```ts
// script setup — 关键片段

// 1. useDetail 取出 handleClose
const { handleShowDetail, handleClose, DetailDialog } = useDetail()

// 2. useCurd 取出 handleAdd
const { handleAdd, handleUpdate, handleDelete, CurdPageDialog } = useCurd({
  formConfig, formData, handleTableRefresh, loading,
})

// 3. 存储行 ID + 包装函数
const tempRowId = ref('')
const showDetail = (rowId: string, code: string) => {
  tempRowId.value = rowId
  handleShowDetail(rowId, code)
}

// 4. 复制核心函数
const copyRecord = (type: 'all' | 'head') => {
  if (!tempRowId.value) return
  http.get(`/materialInbound/selectById/${tempRowId.value}`).then(res => {
    if (!res.data) return
    handleClose()
    handleAdd('/materialInbound/add', beforeConfirm, true)
    Object.keys(formData).forEach(key => {
      ;(formData as any)[key] = (res.data as any)[key]
    })
    formData.orderId = undefined
    formData.orderDate = Date.now()
    if (type === 'head') {
      formData.detailList = []
    }
  })
}

// 5. 列中点击单据号查看详情，使用 showDetail 包装
// onClick={() => showDetail(row.orderId, '1001034')}
```

## 适配要点

### formData 字段名因页面而异

每个页面的主键 ID 和日期字段名可能不同，需要对应修改：

| 页面类型 | 主键 ID 字段 | 日期字段 |
|---------|-------------|---------|
| 材料入库 | `orderId` | `orderDate` |
| 报销单 | `reimId` | `reimDate` |
| 出库单 | `orderId` | `orderDate` |

```ts
formData.orderId = undefined    // 替换为实际主键字段名
formData.orderDate = Date.now()  // 替换为实际日期字段名
```

### API 路径

- 查询详情：`/<baseUrl>/selectById/${rowId}`
- 新增提交：`/<baseUrl>/add`

### 额外初始化逻辑

某些页面打开新增弹窗前需要额外操作，插入在 `handleAdd` 之前：

```ts
// 例：后勤入库需要根据日期加载物料列表
loadMaterials(res.data.orderDate)
handleAdd('/logisticsInbound/add', beforeConfirm, true)
```

### 明细行预处理

如果原单据的明细列表有计算字段（如 `moneyCount = money + tax`），整单复制时可能需要在拷贝后遍历处理：

```ts
if (type !== 'head') {
  formData.detailList.forEach(item => {
    item.moneyCount = (item.money || 0) + (item.valueAddedTax || 0)
  })
}
```

### TS 类型错误

`Object.keys(formData).forEach(key => formData[key] = ...)` 在 strict 模式下会报 TS7053。fix：`(formData as any)[key]`。

## hooks-nnw（迁入版）与 hooks（原有）的 useDetail 差异

hooks-nnw 是从南泥湾成本管控项目迁入，其 useDetail 与项目原有 hooks 的 useDetail API 不同：

| 差异点 | hooks-nnw/useDetail | hooks/useDetail |
|--------|-------------------|----------------|
| 关闭方法 | `handleDialogClose()` | `handleClose()` |
| 自定义宽度 | `handleShowDetail(id, code, title, show, width)` 第5参 | 不内置，按 `dialogAttr` |
| 按钮可见性控制 | `showSlot` ref（第4参 `show`） | `isOnlyCustomFooter` ref（第4参） |
| DetailDialog 容器 | `PageDialog`（header 插槽放按钮） | `el-dialog`（footer 插槽放按钮） |

结果一样：详情弹窗底部出现复制按钮。
