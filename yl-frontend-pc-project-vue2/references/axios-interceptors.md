# API 请求封装

## axios 实例

路径：`src/utils/axios.js`

```js
import axios from 'axios'
import store from '@/store'
import router from '@/router'
import { Noty } from 'noty'

const api = axios.create({
  baseURL: process.env.VUE_APP_BASE_URL,
  timeout: 90000,
})

// 请求拦截器：注入 Token
api.interceptors.request.use(config => {
  const token = store.state.user.token
  if (token) {
    config.headers['Authorization'] = `Bearer ${token}`
  }
  return config
})

// 响应拦截器：业务码校验 + 错误处理
api.interceptors.response.use(
  response => {
    const { code, message, data } = response.data

    // 业务成功码范围：20000 ~ 29999
    if (code >= 20000 && code <= 29999) {
      return data ?? response.data
    }

    // 业务失败
    new Noty({ type: 'error', text: message || '请求失败' }).show()
    return Promise.reject(response.data)
  },
  error => {
    const { status } = error.response || {}

    if (status === 401) {
      // Token 过期 → 清除登录状态 → 跳转登录页
      store.dispatch('user/logout')
      router.push('/login')
      return Promise.reject(error)
    }

    if (status === 400 || status === 403 || status === 500) {
      new Noty({ type: 'error', text: `请求错误 ${status}` }).show()
    }

    return Promise.reject(error)
  }
)

export default api
```

---

## 关键设计

| 机制 | 说明 |
|------|------|
| baseURL | 从环境变量 `VUE_APP_BASE_URL` 读取（开发为 `/api`，生产为 `/DTCB`） |
| 超时 | 90 秒（后端某些报表查询可能较慢） |
| Token 注入 | 请求拦截器自动附加 `Authorization: Bearer xxx` |
| 业务码校验 | `code` 不在 20000~29999 区间则视为失败，用 noty 报错 |
| 401 处理 | 自动清除登录状态并跳转登录页 |
| 4xx/5xx 处理 | noty 错误提示 |

---

## 组件中调用方式

### 方式一：通过 Mixins 封装方法（推荐）

```js
// settingMixin 提供
this.getDataList('/back/workGroup/findList', searchForm)
this.addOrUpdate(urls, formData)
this.deleteItem('/back/workGroup/del/123')
this.getOptions('get', '/back/workGroup/findOne', { id: 123 })
```

### 方式二：直接调用 axios 实例

```js
import api from '@/utils/axios'

api({ url: '/oauth/token', method: 'post', auth: { username, password } })
  .then(res => { /* ... */ })

api.post('/back/workGroup/add', formData)
  .then(res => { /* ... */ })
```

---

## 错误处理

由于项目使用 `noty` 库（而非 Element UI 的 `this.$message`），错误提示方式如下：

```js
import { Noty } from 'noty'

new Noty({
  type: 'error',    // success | error | warning | info
  text: '操作失败',
  timeout: 3000,
}).show()
```

项目也封装了 `this.$notice()` 方法（通过 Vue 插件 `src/utils/vue-notice.js`），简化调用：

```js
this.$notice({ type: 'success', text: '操作成功' })
```
