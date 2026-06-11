# Vuex 权限与状态管理

## Store 结构

```
store/
├── index.js      # 创建 Vuex.Store，自动注册 modules
├── getters.js    # 全局 getters
└── modules/
    ├── user.js   # 用户 + token + 菜单
    ├── system.js # 窗口尺寸 + 滚动条宽度
    └── finance.js # 财务日期（年/月/日）
```

---

## index.js — Store 入口

```js
import Vue from 'vue'
import Vuex from 'vuex'
import getters from './getters'

Vue.use(Vuex)

// 自动注册 modules 目录下所有 .js 文件
const modulesFiles = require.context('./modules', true, /\.js$/)
const modules = modulesFiles.keys().reduce((modules, modulePath) => {
  const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
  modules[moduleName] = modulesFiles(modulePath).default
  return modules
}, {})

const store = new Vuex.Store({
  modules,
  getters,
  plugins: [
    createPersistedState({
      storage: window.localStorage,
      reducer: state => ({
        user: state.user,       // 持久化用户模块
        system: state.system,   // 持久化系统模块
        // finance 不持久化（每次启动重新获取日期）
      }),
    }),
  ],
})

export default store
```

---

## Modules 详解

### user 模块

| state | 类型 | 说明 |
|-------|------|------|
| token | string | 登录 Token |
| token_type | string | Token 类型 |
| userInfo | object | 用户信息（含 isAdmin） |
| userMenu | array | 用户菜单列表 |

| actions | 说明 |
|---------|------|
| login | 登录 → 保存 token + userInfo + userMenu |
| logout | 退出 → 清除所有状态 |
| updateUserInfo | 更新用户信息 |
| updateUserMenu | 更新菜单 |

### system 模块

| state | 类型 | 说明 |
|-------|------|------|
| rect | string | 窗口宽高（`WIDTHxHEIGHT`） |
| scrollBarWidth | number | 滚动条宽度 |

| actions | 说明 |
|---------|------|
| setRect | 设置窗口尺寸 |
| setScrollBarWidth | 设置滚动条宽度 |

### finance 模块

| state | 类型 | 说明 |
|-------|------|------|
| year | number | 当前财年 |
| month | number | 当前月份 |
| day | number | 当前日期 |

| actions | 说明 |
|---------|------|
| setFinanceDate | 设置财务日期 |

> finance 模块**不被持久化**，每次应用启动时重新从后端获取当前财务日期。

---

## 组件中使用

```js
// 读取 state
this.$store.state.user.token
this.$store.state.user.userInfo

// 读取 getters
this.$store.getters.token
this.$store.getters.userInfo

// 调用 action
this.$store.dispatch('login', { username, password })
this.$store.dispatch('logout')

// 辅助函数
import { mapState, mapActions } from 'vuex'

export default {
  computed: {
    ...mapState('user', ['token', 'userInfo']),
  },
  methods: {
    ...mapActions('user', ['login', 'logout']),
  },
}
```

---

## 全局 Getters

```js
// store/getters.js
export default {
  token: state => state.user.token,
  access_token: state => state.user.token,
  userInfo: state => state.user.userInfo,
  userMenu: state => state.user.userMenu,
  rect: state => state.system.rect,
  scrollBarWidth: state => state.system.scrollBarWidth,
}
```

---

## vuex-persistedstate 配置

```js
import createPersistedState from 'vuex-persistedstate'

createPersistedState({
  storage: window.localStorage,
  reducer: state => ({
    user: state.user,       // 持久化
    system: state.system,   // 持久化
    // finance 不持久化
  }),
})
```
