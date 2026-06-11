# Vue 2 核心组件

## DataGrid — 自研核心表格

DataGrid 是本项目的核心组件，功能涵盖了 Element UI `el-table` 的所有能力并有所扩展。

### 关键能力

| 能力 | 说明 |
|------|------|
| 固定列 | 支持左右固定列 |
| 行合并 (rowspan) | 指定列按相同值合并行 |
| 行编辑模式 | 点击行进入编辑态 |
| 多选 | checkbox 列，支持跨页保持勾选 |
| 汇总行 | 表格底部合计行 |
| 虚拟滚动 | 大数据量下的性能优化 |
| 自适应高度 | 根据窗口高度自动调整 |

### 基本用法

```vue
<DataGrid
  :options="gridOptions"
  :data="tableData"
  :loading="loading"
  @search="handleSearch"
  @select-change="handleSelectChange"
/>
```

### gridOptions 配置

```js
gridOptions: {
  columns: [
    { prop: 'name', label: '名称', width: 120, fixed: 'left' },
    { prop: 'code', label: '编码', width: 100 },
    { prop: 'status', label: '状态', width: 80, formatter: (val) => statusMap[val] },
  ],
  search: {
    fields: [
      { prop: 'name', label: '名称', type: 'input' },
      { prop: 'status', label: '状态', type: 'select', options: statusOption },
    ],
  },
  selection: true,       // 开启多选
  rowKey: 'id',          // 行唯一标识
  editMode: 'row',       // 行编辑模式
}
```

---

## Panel — 内容面板

```vue
<Panel title="基本信息" :loading="loading">
  <!-- 任意内容 -->
</Panel>
```

带装饰角、loading 状态的面板容器。

---

## Dialog — 弹窗

```vue
<Dialog
  :visible.sync="dialogVisible"
  title="新增"
  width="800px"
  @confirm="handleConfirm"
>
  <!-- 表单内容 -->
</Dialog>
```

---

## Pagination — 分页

```vue
<Pagination
  :total="total"
  :current-page="pageNum"
  :page-size="pageSize"
  @change="handlePageChange"
/>
```

---

## 其他常用组件

| 组件 | 用途 |
|------|------|
| Menu | 左侧导航菜单 |
| NavTab | 导航标签页 |
| Select-v2 | 下拉选择器（带搜索、远程搜索） |
| DropDown / DropDownMulti | 下拉选择（单选/多选） |
| DateSelector | 日期选择封装 |
| Button | 按钮（type: update/delete/cancel/submit） |
| FileUpload / PicUpload | 文件/图片上传 |
| Tree | 树形控件 |
| ChartBar / BarCascade / BarMore / BarStereogram | Canvas 自绘图表 |
| RollNumber | 数字滚动效果 |
| Particles | 粒子特效背景 |
| Screenfull | 全屏按钮 |
| UserInfo | 用户信息展示 |
