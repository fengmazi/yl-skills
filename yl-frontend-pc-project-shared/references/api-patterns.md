# API 请求模式

> 本页描述引领体系所有 PC 项目后端的**统一请求规范**，与前端框架版本（Vue 2 / Vue 3）无关。
> 具体框架中的 HTTP 封装方式（axios 实例、Pinia/Vuex 中的 token 注入等）请查阅对应 version skill。

## 分页查询参数格式

项目后端统一使用以下请求参数格式：

```ts
{
  current: 1,           // 当前页
  pageSize: 20,         // 每页条数
  querys: [             // 查询条件
    {
      property: 'name',
      operator: 'CONTAINS',
      value: '测试'
    }
  ],
  sorts: [              // 排序
    {
      property: 'createDate',
      sort: 'DESC'
    }
  ]
}
```

### 操作符表

| 操作符 | 说明 |
|-----|-----|
| EQUAL | 等于 |
| NOT_EQUAL | 不等于 |
| CONTAINS | 包含 |
| NOT_CONTAINS | 不包含 |
| GT | 大于 |
| GTE | 大于等于 |
| LT | 小于 |
| LTE | 小于等于 |
| BETWEEN | 范围 |
| NOT_BETWEEN | 不在范围 |
| IN | 在集合中 |
| NOT_IN | 不在集合中 |
| NULL | 为空 |
| NOT_NULL | 不为空 |
| STARTS_WITH | 开头匹配 |
| ENDS_WITH | 结尾匹配 |

## 常用操作模式

### CRUD 标准写法

```ts
// 新增
http.post('/api/add', formData).then(() => {
  message.success('新增成功')
  handleTableRefresh()
})

// 修改
http.post('/api/update', formData).then(() => {
  message.success('修改成功')
  handleTableRefresh()
})

// 删除（带确认）
confirm('确定删除吗？').then(() => {
  http.delete('/api/delete/id').then(() => {
    message.success('删除成功')
    handleTableRefresh()
  })
})

// async/await 写法
const handleSubmit = async () => {
  try {
    loading.value = true
    await http.post('/api/add', formData)
    message.success('操作成功')
    handleTableRefresh()
  } catch (error) {
    console.error('操作失败:', error)
  } finally {
    loading.value = false
  }
}
```

> 各版本项目中 message/confirm 的导入路径不同：Vue 3 用 `ElMessage`/`ElMessageBox`（element-plus），Vue 2 用 `this.$message`/`this.$confirm` 或 `noty`（element-ui）。以实际项目为准。

### 审批操作

```ts
// 提交审批
http.post('/api/submit', {
  businessId: row.id,
  taskTypeCode: '1001001'
}).then(() => {
  message.success('提交成功')
  handleTableRefresh()
})

// 审批通过/驳回
http.post('/api/check', {
  businessId: row.id,
  taskTypeCode: '1001001',
  approveStatus: 'pass',   // pass: 通过, reject: 驳回
  opinion: '同意'
}).then(() => {
  message.success('审批成功')
  handleTableRefresh()
})

// 撤销
http.post('/api/revocation', {
  businessId: row.id,
  taskTypeCode: '1001001'
}).then(() => {
  message.success('撤销成功')
  handleTableRefresh()
})
```

### 批量操作

#### 模式一：从已加载数据中勾选（常规批量）

```ts
http.post('/api/batch', {
  ids: selectedRows.map(r => r.id)
}).then(() => {
  message.success('提交成功')
  dataTableRef.value?.clearCheckbox()  // 清除勾选
  handleTableRefresh()
})
```

#### 模式二：进入批量模式 → 自动请求符合条件的全部数据（筛选后批量）

**适用场景**：用户点击"批量审批"等按钮后，自动筛选符合条件的数据（如所有 `approveStatus === 'submit'` 的记录）。

**核心思路**：向 `queryList` 注入过滤条件 → 表格仅展示符合条件的数据 → 用户勾选后提交 → 完成后移除过滤条件恢复原始数据。

1. 进入批量模式时注入过滤条件：
```ts
queryList.value = [...queryList.value.filter(
  (q: any) => q.property !== 'approveStatus'
)]
queryList.value.push({
  property: 'approveStatus',
  value: 'submit',
  operator: 'EQUAL',
})
handleQueryPage(pageNo.value, pageSize.value, queryList.value, [])
```

2. 取消/完成后移除过滤条件：
```ts
queryList.value = [...queryList.value.filter(
  (q: any) => q.property !== 'approveStatus'
)]
handleQueryPage(pageNo.value, pageSize.value, queryList.value, [])
```

> 具体实现（工具栏按钮 TSX 渲染、弹窗确认 UI）因 Vue 版本和 UI 库不同而异，以各 version skill 和项目实际代码为准。

### 文件下载

```ts
import { downloadFile } from '@/utils'

// POST 下载
downloadFile('/api/export', '文件名.xlsx', 'post', { params: {} })
// GET 下载
downloadFile('/api/download/file', '文件名.pdf', 'get')
```

### 关联数据 / 下拉选项查询

```ts
// 查询关联明细
http.post('/api/detail/query', { masterId: row.id }).then(res => {
  detailData.value = res.data
})

// 查询下拉选项（映射为 { label, value } 格式）
http.post('/api/options').then(res => {
  getConfig('field').options = res.data.map(item => ({
    label: item.name,
    value: item.id
  }))
})
```

## useTable 请求流程

`useTable`（Vue 3 hooks）或 settingMixin（Vue 2 mixins）内部封装了分页请求，将 `pageNo`、`pageSize`、`queryList`、`sortList` 组装为后端要求的分页参数格式。页面只需调用 `handleTableRefresh()` 或 `handleQueryPage()` 即可触发请求。

> Vue 3 各项目（`-bt` vs `-nnw` 后缀）的 `useTable` 实现可能有差异，Vue 2 项目使用 Mixins 方式。以当前项目实际代码为准。
