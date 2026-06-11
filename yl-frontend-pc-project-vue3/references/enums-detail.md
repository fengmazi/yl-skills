# 枚举体系详解

## 双轨制

| 类型 | 存放位置 | 获取方式 | 适用场景 |
|------|---------|---------|---------|
| 静态枚举 | `src/enums/` 或 `src/enums-bt/` 目录 | 直接 import enums | 前端固定选项（状态、是否、性别等） |
| 动态枚举 | 服务端维护 | `useEnum(groupNo)` | 业务枚举（费用类型、单据类别等） |

---

## 静态枚举加载机制

目录下的 `.ts` 文件会被自动 glob 加载，**每个文件自动生成三个导出**：

```ts
// 假设 src/enums/status.ts：
export default {
  U: '启用',
  S: '停用'
}
```

自动生成：

| 导出 | 结构 | 用途 |
|------|------|------|
| `enums.statusOption` | `[{ value: 'U', label: '启用' }, ...]` | select/radio 的 options |
| `enums.statusMap` | `{ U: '启用', S: '停用' }` | 值→标签映射，formatter 显示 |
| `enums.tagTypeMap` | `{ U: 'success', S: 'info' }` | el-tag 的 type 属性 |

**注意**：`tagTypeMap` 的值对应 Element Plus 的标签类型：`success` / `warning` / `danger` / `info`。

---

## 枚举使用惯式

### 表单中使用

```ts
{
  type: 'select',
  prop: 'status',
  name: '状态',
  options: enums.statusOption
}
```

### 表格筛选中使用

```ts
{
  field: 'status',
  title: '状态',
  filterParam: {
    type: Array,
    options: enums.statusOption
  }
}
```

### 表格列中显示（带标签）

```ts
{
  field: 'status',
  title: '状态',
  slots: {
    default: ({ row }) => [
      <el-tag type={enums.tagTypeMap[row.status]}>
        {enums.statusMap[row.status]}
      </el-tag>
    ]
  }
}
```

### 纯文本格式化

```ts
{
  field: 'isTax',
  title: '是否含税',
  formatter: ({ cellValue }) => enums.booleanMap[cellValue]
}
```

### 详情中显示

```tsx
// 文本显示
<span>{enums.statusMap[detail.value.status]}</span>

// 标签显示
<el-tag type={enums.tagTypeMap[detail.value.status]}>
  {enums.statusMap[detail.value.status]}
</el-tag>
```

---

## 安全回退

代码值转中文标签时，使用 `||` 回退到原值：

```ts
// 值不存在于 map 中时，显示原始值
enums.statusMap[value] || value
```

---

## 动态枚举

```ts
import useEnum from '@/hooks-bt/useEnum'  // 或 '@/hooks/useEnum'

const { getEnum } = useEnum()

getEnum(2002).then(res => {
  // res.options = [{ label: 'xxx', value: 'xxx' }]
  getConfig('field').options = res.options
})
```

**缓存机制**：`useEnum` 内部对相同 `groupNo` 有缓存，不会重复请求。

---

## 常见枚举命名

| 枚举名 | Option | Map | 常见用途 |
|--------|--------|-----|---------|
| status | statusOption | statusMap | 通用启用/停用 |
| boolean | booleanOption | booleanMap | 是/否 (Y/N) |
| approveStatus | approveStatusOption | approveStatusMap | 审批状态（新建/提交/审核中/通过/驳回/撤销/弃审） |
| orderStatus | orderStatusOption | orderStatusMap | 订单状态 |
| sex | sexOption | sexMap | 性别 |

各项目可能有不同的枚举定义，以实际 `src/enums/` 或 `src/enums-bt/` 目录为准。
