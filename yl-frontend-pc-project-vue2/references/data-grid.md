# DataGrid 详解

DataGrid 是本项目的核心自研表格组件（非 Element UI 的 `el-table`）。

## 关键能力

| 能力 | 说明 | 配置 |
|------|------|------|
| 固定列 | 左右固定列不随横向滚动 | `fixed: 'left'` / `fixed: 'right'` |
| 行合并 | 相邻行同值列自动合并（rowspan） | `merge: true` + `mergeField: 'col'` |
| 行编辑 | 点击行进入编辑态，行内修改 | `editMode: 'row'` |
| 多选 | checkbox 列，支持跨页保持勾选状态 | `selection: true` |
| 汇总行 | 表格底部自动计算合计 | `summary: true` + `summaryFields: ['amount']` |
| 虚拟滚动 | 大数据量渲染优化 | `virtualScroll: true` |
| 搜索区域 | 内置搜索表单（可折叠） | `search.fields: [...]` |
| 排序 | 点击列头排序 | `sortable: true` |
| 单元格渲染 | 自定义 formatter / slot | `formatter: fn` 或 `<template #colName>` |
| 导出 | 导出 Excel | `export: true` |

---

## gridOptions 完整配置

```js
gridOptions: {
  // 列定义
  columns: [
    {
      prop: 'name',           // 字段名（必填）
      label: '名称',           // 列标题（必填）
      width: 150,             // 列宽
      minWidth: 100,          // 最小列宽
      fixed: 'left',          // 固定列：'left' | 'right'
      sortable: true,         // 可排序
      formatter: (val, row) => statusMap[val],  // 格式化函数
      align: 'center',        // 对齐方式
      merge: true,            // 同行合并
      editable: false,        // 行编辑模式下是否可编辑
    },
  ],

  // 多选配置
  selection: true,            // 开启多选
  rowKey: 'id',               // 行唯一标识（多选必须）

  // 行编辑
  editMode: 'row',            // 行编辑模式

  // 汇总行
  summary: true,
  summaryFields: ['amount', 'quantity'],  // 需要合计的字段
  summaryMethod: ({ columns, data }) => { /* 自定义合计逻辑 */ },

  // 搜索区域
  search: {
    fields: [
      {
        prop: 'name',         // 搜索字段名
        label: '名称',         // 标签
        type: 'input',        // 输入类型：input | select | date | daterange
        options: [],          // select 类型时的选项
        placeholder: '请输入',
      },
    ],
    collapsed: false,         // 是否默认折叠
  },

  // 导出
  export: {
    visible: true,
    filename: '导出文件',
  },
}
```

---

## 主要事件

| 事件 | 参数 | 说明 |
|------|------|------|
| `@search` | `searchForm` | 搜索按钮点击 |
| `@select-change` | `selectedRows` | 多选变化 |
| `@sort-change` | `{ prop, order }` | 排序变化 |
| `@row-click` | `row` | 行点击 |
| `@cell-click` | `row, column, cell` | 单元格点击 |

---

## 与 vxe-table（Vue 3 版 DataTable）的对照

| 功能 | Vue 2 DataGrid | Vue 3 DataTable（vxe-grid） |
|------|---------------|---------------------------|
| 表格库 | 自研 Canvas 渲染 | vxe-table |
| 配置方式 | `gridOptions` 对象 | `Column[]` + props |
| 多选 | `selection: true` | `checkboxConfig` |
| 行编辑 | `editMode: 'row'` | `editConfig.mode: 'row'` |
| 汇总行 | `summary: true` | `footerConfig` + `footerMethod` |
| 固定列 | `fixed: 'left'` | `fixed: 'left'`（用法一致） |
| 空值显示 | 组件内置 | `formatter` 中 `|| '-'` |

> 两种表格的具体 API 不同，但设计理念一致：配置驱动，列定义声明式。
