# 常用工具函数

> 以下函数路径可能因项目后缀而异（`@/utils` vs `@/utils-bt`），以当前项目实际导入路径为准。

## 金额格式化

### formatMoney

将数字转换为千分位格式：

```ts
import { formatMoney } from '@/utils'

formatMoney(10000)        // "10,000.00"
formatMoney(1234567.89)   // "1,234,567.89"

// 表格列中使用
{
  field: 'money',
  title: '金额',
  attrs: { formatMoney: true, precision: 2 },
  formatter: ({ cellValue }) => formatMoney(cellValue)
}
```

### smalltoBIG

数字转大写金额：

```ts
import { smalltoBIG } from '@/utils'
smalltoBIG(1234.56)  // "壹仟贰佰叁拾肆元伍角陆分"
```

---

## 权限检查

### checkAuth

检查用户是否有某个按钮权限：

```ts
import { checkAuth } from '@/utils'

// 在 JSX 中使用
checkAuth('xxx-add') && <el-button onClick={...}>新增</el-button>
```

---

## 文件下载

```ts
import { downloadFile, downloadStaticFile } from '@/utils'

// 接口下载（POST）
downloadFile('/api/export', '文件名.xlsx', 'post', { params: {} })

// 接口下载（GET）
downloadFile('/api/download', '文件.pdf', 'get')

// URL / Blob 下载
downloadStaticFile('https://example.com/file.pdf', '文件.pdf')
downloadStaticFile(blob, '文件.xlsx')
```

---

## 树形数据转换系列

项目中大量使用树形数据（部门、费用项目等），这些工具函数提供统一的数据格式转换。

### treeDataToKeyValue

将树形数据转换为树形下拉选项格式：

```ts
import { treeDataToKeyValue } from '@/utils'

const options = treeDataToKeyValue(data, 'deptId', 'deptName')
// → [{ id: '1', label: '部门1', children: [...] }, ...]

// 带禁用判断
const options = treeDataToKeyValue(data, 'deptId', 'deptName', 'children', (item) => item.status === 'S')
```

### treeDataToFlatOption

将树形数据扁平化为下拉选项：

```ts
import { treeDataToFlatOption } from '@/utils'

const options = treeDataToFlatOption(data, 'deptId', 'deptName')
// → [{ key: '1', value: '部门1' }, { key: '1-1', value: '子部门1' }]
```

### treeDataToMap

将树形数据转换为 ID-名称 Map：

```ts
import { treeDataToMap } from '@/utils'

const map = treeDataToMap(data, 'deptId', 'deptName')
// → { '1': '部门1', '2': '部门2' }
```

**注意**：迁移版本可能有变体如 `treeDataToKeyValueBt`（同时处理 `disabled` 和 `isDisabled` 字段），以实际项目代码为准。

---

## 计算公式

### calculateFormula

运行时计算公式，支持 `data["key"]` 引用：

```ts
import { calculateFormula } from '@/utils'

const result = calculateFormula('data["price"] * data["quantity"]', {
  price: 100,
  quantity: 5
})  // 500

// 支持 ?? 默认值
calculateFormula('(data["price"] ?? 0) * data["quantity"]', { quantity: 5 })  // 0
```

---

## 日期处理

使用 `@vueuse/core` 的 `useDateFormat`：

```ts
import { useDateFormat } from '@vueuse/core'

// 格式化
useDateFormat(Date.now(), 'YYYY-MM-DD').value        // "2024-01-15"
useDateFormat(Date.now(), 'YYYY-MM-DD HH:mm:ss').value // "2024-01-15 10:30:00"

// 表格中使用（区分毫秒/秒时间戳）
{
  field: 'createDate',
  title: '创建时间',
  formatter: ({ cellValue }) => cellValue
    ? useDateFormat(cellValue, 'YYYY-MM-DD').value
    : ''
}
```

---

## 流程节点构建

### buildFlowNodeStepsData

构建流程节点步骤数据，常用于详情页的步骤条展示：

```ts
import { buildFlowNodeStepsData } from '@/utils'

const steps = buildFlowNodeStepsData(data)
// → [{ name: '上会', step: 'M', active: {...}, current: false, jump: true }, ...]
```

---

## 其他工具函数

| 函数 | 用途 |
|------|------|
| `getParam(name)` | 获取浏览器 URL 参数 |
| `randomContent(n)` / `randomString(n)` | 生成随机字符串 |
| `getNextZIndex()` | 获取递增的 z-index（用于弹窗层叠） |
| `getAuthFromNavTree(data)` | 从导航树提取路由权限列表 |
| `calcFooter(columns, tableData)` | 计算表格合计行数据 |
