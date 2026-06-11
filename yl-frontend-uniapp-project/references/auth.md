# uni-app 认证与权限

## 登录流程

### OAuth2 密码模式

```ts
import { http } from '@/utils/http'

const handleLogin = async (username: string, password: string) => {
  const res = await http({
    url: '/oauth/token',
    method: 'POST',
    data: {
      grant_type: 'password',
      username,
      password,
    },
  })
  // res: { access_token, token_type, userInfo, menu }
  appStore.setToken(res.access_token)
  appStore.setUserInfo(res.userInfo)
  appStore.setMenu(res.menu)
}
```

### 钉钉登录集成（部分项目）

```ts
import dd from 'dingtalk-jsapi'

const handleDingLogin = () => {
  dd.ready(() => {
    dd.runtime.permission.requestAuthCode({
      corpId: 'xxx',
      onSuccess: (result) => {
        const authCode = result.code
        // 用 authCode 换取后端 token
        http({ url: '/oauth/dingLogin', data: { authCode } })
          .then(res => {
            appStore.setToken(res.access_token)
          })
      },
    })
  })
}
```

---

## Pinia Store

所有项目使用单一 `app` store 管理认证状态：

```ts
// stores/modules/app.ts
import { defineStore } from 'pinia'

export const useAppStore = defineStore('app', {
  state: () => ({
    token: '',
    userInfo: null as any,
    auth: [] as string[],
    menu: [] as any[],
    deptInfo: null as any,
  }),

  actions: {
    setToken(token: string) { this.token = token },
    setUserInfo(info: any) { this.userInfo = info },
    setMenu(menu: any[]) { this.menu = menu },
    setAuth(auth: string[]) { this.auth = auth },

    logout() {
      this.token = ''
      this.userInfo = null
      this.auth = []
      this.menu = []
    },
  },

  persist: {
    storage: {
      getItem: (key: string) => uni.getStorageSync(key),
      setItem: (key: string, value: any) => uni.setStorageSync(key, value),
    },
  },
})
```

> 使用 `pinia-plugin-persistedstate` 持久化到 `uni.storageSync`（uni-app 的本地存储 API）。

---

## 权限检查

```ts
// utils/index.ts
import { useAppStore } from '@/stores/modules/app'

export function checkAuth(permission: string): boolean {
  const appStore = useAppStore()
  return appStore.auth.includes(permission)
}

// 页面中使用
if (checkAuth('task-add')) {
  // 显示新增按钮
}
```

---

## 自动登录检查

```ts
// App.vue onLaunch
onLaunch(() => {
  const appStore = useAppStore()
  if (appStore.token) {
    // 已有 token，验证是否过期
    http({ url: '/system/user/info' })
      .then(res => {
        appStore.setUserInfo(res)
      })
      .catch(() => {
        appStore.logout()
        uni.reLaunch({ url: '/pages/login/login' })
      })
  } else {
    uni.reLaunch({ url: '/pages/login/login' })
  }
})
```
