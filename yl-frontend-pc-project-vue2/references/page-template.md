# Vue 2 页面开发模板

## 文件书写顺序

Vue 2 SFC 标准书写顺序：

```vue
<template>
  <!-- 模板 -->
</template>

<script>
// 组件逻辑
export default {
  name: 'PageName',
  mixins: [],
  components: {},
  data() {},
  computed: {},
  watch: {},
  created() {},
  mounted() {},
  methods: {},
  beforeDestroy() {},
}
</script>

<style lang="scss" scoped>
/* 样式 */
</style>
```

---

## setting 设置页标准模板

最常用的 CRUD 页面模式。以下为完整模板：

```vue
<template>
  <div class="page-container">
    <Panel title="班组信息">
      <DataGrid :options="gridOptions" @search="handleSearch">
        <template #toolbar>
          <Button type="add" @click="handleAdd">新增</Button>
        </template>
      </DataGrid>
    </Panel>

    <Panel title="数据列表">
      <DataGrid
        :options="gridOptions"
        :data="tableData"
        :loading="loading"
        @select-change="handleSelectChange"
      >
        <template #operation="{ row }">
          <Button type="update" @click="handleEdit(row)">编辑</Button>
          <Button type="delete" @click="handleDelete(row.id)">删除</Button>
        </template>
      </DataGrid>
      <Pagination
        :total="total"
        :current-page="pageNum"
        :page-size="pageSize"
        @change="handlePageChange"
      />
    </Panel>

    <Dialog
      :visible.sync="dialogVisible"
      :title="dialogTitle"
      @confirm="handleSubmit"
    >
      <el-form ref="form" :model="formData" label-width="100px">
        <el-form-item label="名称" prop="name">
          <el-input v-model="formData.name" />
        </el-form-item>
        <!-- 更多表单项 -->
      </el-form>
    </Dialog>
  </div>
</template>

<script>
import settingMixin from '@/views/mixins/settingMixin'

export default {
  name: 'TeamInfo',
  mixins: [settingMixin],
  data() {
    return {
      gridOptions: {
        columns: [
          { prop: 'name', label: '名称', width: 150 },
          { prop: 'status', label: '状态', width: 80 },
          { prop: 'createTime', label: '创建时间', width: 160 },
          { label: '操作', width: 150, slot: 'operation' },
        ],
        search: {
          fields: [
            { prop: 'name', label: '名称', type: 'input' },
          ],
        },
      },
    }
  },
  methods: {
    handleSearch(params) {
      this.searchForm = params
      this.loadData()
    },
    loadData() {
      this.getDataList('/back/workGroup/findList', this.searchForm)
    },
    handleSubmit() {
      this.addOrUpdate(
        {
          add: '/back/workGroup/add',
          edit: '/back/workGroup/edit',
        },
        this.formData,
        () => {
          this.dialogVisible = false
          this.loadData()
        }
      )
    },
  },
  created() {
    this.loadData()
  },
}
</script>
```

---

## 文件命名约定

| 类型 | 命名 | 示例 |
|------|------|------|
| 页面目录 | 小驼峰 | `teamInfo/`, `costCenter/` |
| 页面入口 | index.vue | `views/teamInfo/index.vue` |
| 组件文件 | 大驼峰 .vue | `DataGrid.vue`, `Panel.vue` |
| Mixins | 小驼峰 + Mixin 后缀 | `settingMixin.js` |
| 工具函数 | 小驼峰 | `axios.js`, `math.js` |

---

## 搜索表单 → querys 转换

`searchToQuerys` 工具函数将搜索表单转换为后端要求的 `querys[]` 格式：

```js
import { searchToQuerys } from '@/utils'

// 输入
const searchForm = { name: '测试', status: 'U' }
// 输出
const querys = [
  { property: 'name', operator: 'CONTAINS', value: '测试' },
  { property: 'status', operator: 'EQUAL', value: 'U' },
]
```

---

## transition 动画

Vue 2 页面切换使用 `<transition>` 包裹 `<router-view>`，通常配合 `keep-alive`：

```vue
<transition name="fade-transform" mode="out-in">
  <keep-alive>
    <router-view />
  </keep-alive>
</transition>
```
