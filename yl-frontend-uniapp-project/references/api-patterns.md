# uni-app API 请求模式

## http.ts — 请求封装

所有项目使用统一的 `uni.request` 封装，路径：`src/utils/http.ts`。

### 核心实现

```ts
// utils/http.ts
const BASE_URL = import.meta.env.VITE_APP_BASE_API || '/api'

interface RequestOptions {
  url: string
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE'
  data?: any
  header?: any
  timeout?: number
}

export function http(options: RequestOptions): Promise<any> {
  return new Promise((resolve, reject) => {
    uni.request({
      url: BASE_URL + options.url,
      method: options.method || 'POST',
      data: options.data,
      header: {
        'Content-Type': 'application/json',
        ...options.header,
      },
      timeout: options.timeout || 10000,
      success: (res) => {
        const { code, message, data } = res.data as any
        if (code >= 20000 && code <= 29999) {
          resolve(data ?? res.data)
        } else {
          uni.showToast({ title: message || '请求失败', icon: 'none' })
          reject(res.data)
        }
      },
      fail: (err) => {
        uni.showToast({ title: '网络异常', icon: 'none' })
        reject(err)
      },
    })
  })
}
```

---

## 全局拦截器 — uni.addInterceptor

在 `main.ts` 中注册全局请求拦截器：

```ts
// main.ts
uni.addInterceptor('request', {
  invoke(args) {
    // 请求前：注入 Token
    const appStore = useAppStore()
    if (appStore.token) {
      args.header = {
        ...args.header,
        Authorization: `Bearer ${appStore.token}`,
      }
    }
  },
  success(res) {
    // 响应成功：检查 401
    if (res.statusCode === 401) {
      const appStore = useAppStore()
      appStore.logout()
      uni.reLaunch({ url: '/pages/login/login' })
    }
  },
  fail(err) {
    console.error('请求失败:', err)
  },
})
```

---

## 条件编译的 baseURL

使用条件编译区分不同平台的 API 地址：

```ts
// #ifdef H5
const BASE_URL = '/api'
// #endif

// #ifdef APP-PLUS
const BASE_URL = 'https://api.example.com'
// #endif
```

---

## 请求队列管理（预算宝塔、车辆项目）

部分项目在 `http.ts` 中实现了请求队列 `taskList`，支持取消重复请求：

```ts
const taskList: (() => void)[] = []

// 添加 abort 能力
const controller = new AbortController()
taskList.push(() => controller.abort())

// 页面卸载时取消所有未完成请求
export function clearTaskList() {
  taskList.forEach(abort => abort())
  taskList.length = 0
}

// 页面 onUnload 中
onUnload(() => {
  clearTaskList()
})
```

---

## 页面内调用方式

API 请求直接在各页面中内联调用，**无独立的 `src/api/` 目录**：

```vue
<script setup lang="ts">
import { http } from '@/utils/http'

// 获取列表
const loadList = async () => {
  const res = await http({
    url: '/back/task/findList',
    data: { current: 1, pageSize: 20, querys: [] },
  })
  list.value = res.records
}

// 提交表单
const handleSubmit = async () => {
  await http({
    url: '/back/task/add',
    method: 'POST',
    data: formData,
  })
}
</script>
```

---

## 错误处理

```ts
try {
  await http({ url: '/api/submit', data: formData })
  uni.showToast({ title: '提交成功', icon: 'success' })
} catch (err) {
  uni.showToast({ title: '提交失败', icon: 'error' })
}
```
