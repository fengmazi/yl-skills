# Vue 2 架构

## Options API + Mixins + Filters

Vue 2 项目使用经典的 Options API 风格，不涉及 Composition API 或 TypeScript。

### 代码复用体系

| 方式 | 用途 | 示例 |
|------|------|------|
| **Mixins** | 共享 data/methods/生命周期 | `settingMixin`（CRUD）、`chartMixin`（图表自适应） |
| **Vuex Store** | 全局状态管理 | user（token/菜单）、system（窗口尺寸）、finance（日期） |
| **Filters** | 模板格式化 | `{{ value | formatDate }}` |
| **Components** | UI 复用 | DataGrid、Panel、Dialog、Pagination |

### 组件编写模式

```vue
<template>
  <div class="page-container">
    <Panel title="页面标题">
      <!-- 搜索区域 -->
      <DataGrid :options="gridOptions" @search="handleSearch" />
    </Panel>
    <Panel title="数据列表">
      <DataGrid :options="gridOptions" :data="tableData" />
      <Pagination :total="total" @change="handlePageChange" />
    </Panel>
  </div>
</template>

<script>
import settingMixin from '@/views/mixins/settingMixin'

export default {
  name: 'TeamInfo',
  mixins: [settingMixin],
  data() {
    return {
      gridOptions: { /* ... */ },
    }
  },
  methods: {
    handleSearch(params) {
      this.getDataList('/back/workGroup/findList', params)
    },
  },
}
</script>
```

### Mixins 注入原则

- `settingMixin` 提供完整 CRUD 能力（`addOrUpdate`、`deleteItem`、`getDataList`、分页、导入导出）
- `chartMixin` 提供 ECharts 自适应 resize + keep-alive 激活处理
- baseMixin（`mixins/index.js`）提供 Panel + DataGrid + Menu 组件引用 + `getData`/`postData` 基础方法
- Mixins 中的方法通过 `this.xxx()` 直接调用

### 全局 Filters

定义在 `src/filters/` 中，在 `main.js` 中通过 `Vue.filter()` 全局注册，模板中使用管道语法 `{{ value | filterName }}`。

---

## 自研组件体系

本项目对 Element UI 依赖较轻（主要用于表单控件和 loading），核心组件全部自研：

| 组件 | 用途 | 替代 |
|------|------|------|
| DataGrid | 表格（核心） | 替代 el-table |
| Panel | 内容面板容器 | 替代 el-card |
| Dialog | 弹窗 | 替代 el-dialog |
| Pagination | 分页 | 替代 el-pagination |
| Select-v2 | 下拉选择 | 替代 el-select |
| DropDown | 下拉菜单 | — |
| Tree | 树形控件 | 替代 el-tree |

---

## 深色主题设计

项目采用暗蓝基调的深色主题，SCSS 变量定义在 `src/styles/variable.scss`，各组件的配色通过引用变量保持一致性。详见 `references/style-dark-theme.md`。
