# 表单配置项详解

## FormItem 完整属性

| 属性 | 类型 | 说明 |
|-----|-----|-----|
| type | string | 表单项类型（见下方列表） |
| prop | string | 字段名，对应 formData 的 key |
| name | string | 标签名称 |
| options | array | 选项列表 |
| show | Function / boolean | 显示条件 |
| attrs | object | 透传给组件的属性 |
| rule | array | 校验规则 |
| formatter | Function | 格式化函数 |
| labelWidth | number / string | 标签宽度 |
| lg | number | 大屏幕栅格占比 |
| sm | number | 小屏幕栅格占比 |
| colspan | number | 跨列数 |

## 完整 type 列表

| type | 控件 | 常用 attrs |
|------|------|-----------|
| `input` | 文本输入 | `placeholder`, `maxlength`, `clearable` |
| `inputNumber` | 数字输入 | `min`, `max`, `precision`, `step` |
| `password` | 密码输入 | `placeholder` |
| `select` | 下拉选择 | `placeholder`, `clearable`, 数据格式 `{ label, value }` |
| `select-v2` | 下拉选择 v2 | 同 select，性能更好 |
| `treeSelect` | 树形选择 | `checkStrictly`, `clearable`, 需配置 `props={{ disabled, children }}` |
| `treeSelectCheck` | 树形复选框选择（带全选/全不选） | `searchable`, 自带 `multiple` + `show-checkbox` + `checkStrictly` |
| `date` | 日期选择 | `type: 'date'/'daterange'/'datetime'`, `value-format: 'x'` (毫秒), `format: 'YYYY-MM-DD'` |
| `textarea` | 多行文本 | `rows`, `maxlength`, `showWordLimit` |
| `upload` | 文件上传 | `accept`, `limit`, `allowExt`, `fileSize`, `tip` |
| `checkbox` | 复选框 | 数据格式 `{ label, value }` |
| `radio` | 单选框 | 数据格式 `{ label, value }` |
| `cascader` | 级联选择 | `props: { value, label, children }` |
| `richtext` | 富文本 | `height`, `plugins` |
| `text` | 纯文本显示 | — |
| `format` | 格式化显示 | `formatter` |
| `html` | HTML 渲染 | — |
| `empty` | 占位 | — |
| `hr` | 分割线 | — |
| `oneImg` | 单图片 | — |

## 各类型关键细节

### treeSelectCheck 全选/全不选

`treeSelectCheck` 是对 `treeSelect` 的增强版，专用于数据权限等需要"勾选父级自动联动全选所有子级"的场景。

**已内置**：`multiple`、`show-checkbox`、`check-strictly={true}`、`filterable`、`clearable`。
**自定义插槽**：父级节点右侧显示"全选/全不选"按钮，点击后递归勾选/取消所有子孙节点，并通过 `nextTick` + `getCheckedKeys()` 同步 v-model。

**用法示例**（用户管理数据权限弹窗）：

```ts
const perFormConfig = ref([
  [{
    type: 'treeSelect',
    prop: 'deptIds',
    name: '部门',
    options: [],
    attrs: { multiple: true, searchable: true, flat: true },
  }],
  [{
    type: 'treeSelectCheck',
    prop: 'categoryIds',
    name: '存货分类',
    options: [],
    attrs: { searchable: true },
  }],
] as FormItem[][])
```

**实现位置**：`src/components/common/FormItem.vue` 的 `treeSelectCheck` case，约 40 行代码，包含 `handleToggle` 递归逻辑和 `el-tree-select` 的自定义 `default` 插槽。

**依赖的全局样式**（`src/assets/styles/style.scss`）：

```scss
.custom-tree-node {
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  .label { flex: 1; }
  .btn {
    font-size: 0.7rem;
    opacity: 0.3;
    user-select: none;
  }
  &:hover .btn { opacity: 0.8; }
}
```



需配置 `props` 才能正确识别 `disabled` 字段：

```ts
<el-tree-select
  props={{ disabled: 'disabled', children: 'children' }}
  ...
/>
```

**注意**：不同项目（`-bt` vs `-nnw`）的 disabled 字段名可能不同（`disabled` vs `isDisabled` vs `deptStatus`）。

### date 值格式

- `value-format: 'x'` — 毫秒时间戳（推荐，与后端一致）
- `value-format: 'YYYY-MM-DD'` — 日期字符串
- 表格中使用 `useDateFormat(timestamp, 'YYYY-MM-DD')` 显示时，注意区分毫秒/秒

### upload 配置

```ts
{
  type: 'upload',
  prop: 'accList',
  name: '附件',
  attrs: {
    accept: '.pdf,.doc,.docx,.xls,.xlsx',
    limit: 5,
    allowExt: ['pdf', 'doc', 'docx', 'xls', 'xlsx', 'jpg', 'png'],
    fileSize: 10,   // MB
    tip: '支持 PDF、Word、Excel、图片格式，单个文件不超过 10MB'
  }
}
```

### text 纯文本显示

用于只读展示字段，不渲染交互控件：

```ts
{ type: 'text', prop: 'createUserName', name: '创建人' }
```

---

## 动态配置

### 修改选项

```ts
const config = getConfig('fieldName')

// 从接口获取后设置
http.post('/api/options').then(res => {
  config.options = res.data.map(item => ({
    label: item.name,
    value: item.id
  }))
})
```

### 添加校验规则

```ts
const config = getConfig('fieldName')
config.rule = [
  { required: true, message: '请选择', trigger: 'change' }
]
```

### 修改组件属性

```ts
const config = getConfig('amount')
config.attrs = { ...config.attrs, min: 0, precision: 2 }
```

### 条件显示

```ts
// 简单条件
{ type: 'input', prop: 'remark', name: '备注', show: () => formData.type === '1' }

// 复杂条件
{ type: 'input', prop: 'remark', name: '备注', show: computed(() => formData.type === '1' && formData.status === 'U') }
```

---

## 校验规则

### 必填校验

```ts
rule: [
  { required: true, message: '请输入名称', trigger: 'blur' }
]
```

### 自定义校验

```ts
rule: [{
  validator: (rule, value, callback) => {
    if (!value) {
      callback(new Error('请输入金额'))
    } else if (value <= 0) {
      callback(new Error('金额必须大于0'))
    } else {
      callback()
    }
  },
  trigger: 'blur'
}]
```

---

## 表单数据初始化和重置

```ts
const formData = reactive({
  id: undefined,
  name: undefined,
  status: 'U',           // 默认值
  applyDate: new Date().getTime(),  // 默认当前日期
  accList: []            // 附件列表
})

// 重置（打开弹窗前）
Object.assign(formData, {
  id: undefined,
  name: undefined,
  // ...
})
```
