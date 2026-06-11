# Mixins 体系

Vue 2 项目通过 Mixins 实现代码复用，是最核心的编程模式。共 3 个 mixin 文件。

## settingMixin（最重要）

路径：`src/views/mixins/settingMixin.js`

提供完整的 CRUD 操作能力，所有设置类页面（teamInfo、material、costCenter 等 40+ 页面）都使用此 mixin。

### 提供的 Data

```js
{
  dialogVisible: false,     // 弹窗开关
  dialogTitle: '',          // 弹窗标题（新增/编辑）
  formData: {},             // 表单数据
  operateType: '',          // 操作类型（add/edit）
  tableData: [],            // 表格数据
  total: 0,                 // 总条数
  pageNum: 1,               // 当前页
  pageSize: 20,             // 每页条数
  loading: false,           // 加载状态
  searchForm: {},           // 搜索条件
}
```

### 提供的方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `getDataList` | `(url, search, fn)` | POST 分页请求，自动处理 search → querys 转换 |
| `addOrUpdate` | `(urls, data, fn)` | 新增/修改，根据 operateType 判断调哪个接口 |
| `deleteItem` | `(url, fn)` | 删除（会弹出确认框） |
| `getOptions` | `(method, url, data)` | 通用 GET/POST 请求，返回 `res.data.data` |
| `openDialog` | `(title, type)` | 打开弹窗 |
| `handleSearch` | `()` | 触发搜索（pageNum 重置为 1） |
| `handlePageChange` | `(page)` | 分页切换 |
| `handleEdit` | `(row)` | 编辑行（设置 operateType='edit'，打开弹窗） |
| `handleAdd` | `()` | 新增（设置 operateType='add'，打开弹窗） |
| `handleDelete` | `(id)` | 删除确认 + 调接口 |

### 典型页面使用

```vue
<script>
import settingMixin from '@/views/mixins/settingMixin'

export default {
  name: 'TeamInfo',
  mixins: [settingMixin],
  data() {
    return {
      // 页面特有 data
    }
  },
  methods: {
    loadData(searchForm) {
      this.getDataList('/back/workGroup/findList', searchForm)
    },
    handleSubmit() {
      const urls = {
        add: '/back/workGroup/add',
        edit: '/back/workGroup/edit',
      }
      this.addOrUpdate(urls, this.formData, () => {
        this.dialogVisible = false
        this.loadData(this.searchForm)
      })
    },
  },
}
</script>
```

---

## baseMixin — 基础公共 Mixin

路径：`src/views/mixins/index.js`

提供基础能力和组件引用。

### 提供的方法

| 方法 | 说明 |
|------|------|
| `getData(url, fn)` | GET 请求 |
| `postData(url, data, fn)` | POST 请求 |
| `download(url, fileName, method, data)` | 文件下载 |
| `searchToQuerys(searchForm)` | 搜索表单 → `querys[]` 数组转换 |

---

## chartMixin — 图表 Mixin

路径：`src/views/mixins/chartMixin.js`

### 功能

- ECharts 实例的 `resize` 自适应（监听窗口 resize）
- `keep-alive` 激活时重新初始化图表（`deactivated`/`activated` 钩子）
- 统一的图表初始化/销毁流程

### 使用方式

```vue
<script>
import chartMixin from '@/views/mixins/chartMixin'

export default {
  mixins: [chartMixin],
  methods: {
    initChart() {
      this.myChart = this.$echarts.init(this.$refs.chart)
      // ...
    },
  },
}
</script>
```

---

## 多个 Mixins 合并规则

当同时使用多个 mixins 时，按先后顺序合并，**同名的 data/methods 以后面的为准（页面自身的覆盖 mixin）**：

```js
export default {
  mixins: [settingMixin, chartMixin],  // setting 在前，chart 在后
  // 页面自身的 data/methods 优先级最高
}
```
