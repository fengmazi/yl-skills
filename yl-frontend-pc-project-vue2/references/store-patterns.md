# Vuex 状态管理模式

## 模块化规范

所有 Vuex 模块必须使用 `namespaced: true`，避免命名冲突：

```js
// store/modules/user.js
export default {
  namespaced: true,
  state: { /* ... */ },
  mutations: { /* ... */ },
  actions: { /* ... */ },
}
```

---

## 自动模块注册

`store/index.js` 使用 `require.context` 自动扫描 `modules/` 目录，无需手动 import：

```js
const modulesFiles = require.context('./modules', true, /\.js$/)
const modules = modulesFiles.keys().reduce((modules, modulePath) => {
  const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
  modules[moduleName] = modulesFiles(modulePath).default
  return modules
}, {})
```

> 新增模块只需在 `store/modules/` 下创建 `.js` 文件并 `export default` 即可自动注册。

---

## 持久化策略

### 需要持久化的

| 模块 | 原因 |
|------|------|
| user | token、用户信息、菜单在刷新后必须保留 |
| system | 窗口尺寸、滚动条宽度无需每次重新计算 |

### 不需要持久化的

| 模块 | 原因 |
|------|------|
| finance | 财务日期每次启动都应重新从后端获取最新值 |

---

## 常用模式

### 加载状态

```js
// state
loading: false,

// mutation
SET_LOADING(state, val) {
  state.loading = val
},

// action
async fetchData({ commit }, params) {
  commit('SET_LOADING', true)
  try {
    const res = await api.getData(params)
    commit('SET_DATA', res.data)
  } finally {
    commit('SET_LOADING', false)
  }
},
```

### 表单数据管理

```js
// state
formData: {},

// mutation
SET_FORM_DATA(state, key, val) {
  state.formData = { ...state.formData, [key]: val }
},
RESET_FORM(state) {
  state.formData = {}
},
```
