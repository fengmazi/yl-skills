# 枚举双轨制理念

> 本页描述引领体系枚举的**通用设计理念**（双轨制、安全回退），与前端框架版本无关。
> 具体实现（Vue 3 glob 自动加载 vs Vue 2 手动 import）请查阅对应 version skill。

## 双轨制

| 类型 | 存放位置 | 获取方式 | 适用场景 |
|------|---------|---------|---------|
| 静态枚举 | 前端 `src/enums/` 目录 | 直接 import | 前端固定选项（状态、是否、性别等） |
| 动态枚举 | 服务端维护 | 调用枚举接口 | 业务枚举（费用类型、单据类别等） |

## 静态枚举输出规范

每个枚举文件应导出一个对象（key → label 映射），如：

```ts
// src/enums/status.ts
export default {
  U: '启用',
  S: '停用'
}
```

**必须导出的三个格式**（由加载模块自动生成或手动维护）：

| 导出 | 结构 | 用途 |
|------|------|------|
| `xxxOption` | `[{ value: 'U', label: '启用' }, ...]` | select/radio 的 options |
| `xxxMap` | `{ U: '启用', S: '停用' }` | 值→标签映射，formatter 显示 |
| `tagTypeMap`（可选） | `{ U: 'success', S: 'info' }` | 标签颜色类型 |

> **Vue 3**：目录下 `.ts` 文件被 glob 自动加载，自动生成 Option/Map/tagTypeMap 三个导出。
> **Vue 2**：手动 import 每个枚举文件，手动构造 Option/Map。

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

### 表格列中显示

```ts
// 纯文本
{ field: 'status', title: '状态', formatter: ({ cellValue }) => enums.statusMap[cellValue] }

// 带颜色标签（Element Plus el-tag / Element UI el-tag）
// 具体标签组件用法见各 version skill
```

### 详情中显示

```tsx
<span>{enums.statusMap[detail.status]}</span>
```

## 安全回退

代码值转中文标签时，使用 `||` 回退到原值，防止枚举缺失时显示空白：

```ts
enums.statusMap[value] || value
```

## 动态枚举

通过 `useEnum(groupNo)` 或 `this.getOptions()` 从后端获取，内部有缓存机制避免重复请求。

## 常见枚举命名

| 枚举名 | Option | Map | 常见用途 |
|--------|--------|-----|---------|
| status | statusOption | statusMap | 通用启用/停用 |
| boolean | booleanOption | booleanMap | 是/否 (Y/N) |
| approveStatus | approveStatusOption | approveStatusMap | 审批状态 |
| orderStatus | orderStatusOption | orderStatusMap | 订单状态 |
| sex | sexOption | sexMap | 性别 |
