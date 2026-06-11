# 存储与缓存

## uni.storage 体系

| API | 说明 | 类型 |
|-----|------|------|
| `uni.setStorageSync(key, data)` | 同步写入 | 本地存储 |
| `uni.getStorageSync(key)` | 同步读取 | 本地存储 |
| `uni.removeStorageSync(key)` | 同步删除 | 本地存储 |
| `uni.clearStorageSync()` | 同步清空 | 本地存储 |
| `uni.getStorageInfoSync()` | 获取存储信息 | 本地存储 |
| `uni.setStorage({ key, data })` | 异步写入 | 本地存储 |
| `uni.getStorage({ key })` | 异步读取 | 本地存储 |

> 同步方法在 App/H5 端性能较好，小程序端同步操作可能阻塞 UI，大量数据建议用异步方法。

### 存储限制

| 平台 | 限制 |
|------|------|
| H5 | 5-10MB（localStorage） |
| 微信小程序 | 10MB |
| 支付宝小程序 | 10MB |
| App | 不限（原生文件系统） |

---

## Pinia 持久化

### 基础配置

```ts
// main.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)
```

### Store 持久化

```ts
// stores/modules/app.ts
export const useAppStore = defineStore('app', {
  state: () => ({
    token: '',
    userInfo: null as any,
    rememberLoginInfo: { username: '', password: '' },
  }),

  persist: {
    // 适配 uni-app 存储
    storage: {
      getItem: (key: string) => uni.getStorageSync(key),
      setItem: (key: string, value: any) => uni.setStorageSync(key, value),
    },
    // 只持久化部分字段
    pick: ['token', 'rememberLoginInfo'],
  },
})
```

> 敏感信息（如密码）不要持久化，或加密后存储。`rememberLoginInfo` 仅用于"记住密码"功能。

---

## 缓存策略

### 接口数据缓存

```ts
// hooks/useCache.ts
const CACHE_PREFIX = 'cache_'
const CACHE_DURATION = 5 * 60 * 1000  // 5 分钟

interface CacheItem<T> {
  data: T
  timestamp: number
  expire: number
}

export function useCache() {
  const getCache = <T>(key: string): T | null => {
    const raw = uni.getStorageSync(CACHE_PREFIX + key)
    if (!raw) return null

    const item: CacheItem<T> = JSON.parse(raw)
    if (Date.now() - item.timestamp > item.expire) {
      uni.removeStorageSync(CACHE_PREFIX + key)
      return null
    }
    return item.data
  }

  const setCache = <T>(key: string, data: T, duration = CACHE_DURATION) => {
    const item: CacheItem<T> = {
      data,
      timestamp: Date.now(),
      expire: duration,
    }
    uni.setStorageSync(CACHE_PREFIX + key, JSON.stringify(item))
  }

  const clearCache = (key?: string) => {
    if (key) {
      uni.removeStorageSync(CACHE_PREFIX + key)
    } else {
      // 清除所有缓存（只清缓存前缀的）
      const { keys } = uni.getStorageInfoSync()
      keys.filter(k => k.startsWith(CACHE_PREFIX)).forEach(k => {
        uni.removeStorageSync(k)
      })
    }
  }

  return { getCache, setCache, clearCache }
}
```

### 使用示例

```ts
const { getCache, setCache } = useCache()

const loadList = async (forceRefresh = false) => {
  // 优先读缓存
  if (!forceRefresh) {
    const cached = getCache<any[]>('taskList')
    if (cached) {
      list.value = cached
      loading.value = false
      return
    }
  }

  // 请求并缓存
  const res = await http({ url: '/api/task/list' })
  list.value = res.records
  setCache('taskList', res.records)
}
```

---

## 离线策略

### 网络检测

```ts
// 监听网络状态
uni.onNetworkStatusChange(({ isConnected, networkType }) => {
  if (isConnected) {
    // 恢复在线，同步离线数据
    syncOfflineData()
  }
})

// 主动获取当前网络状态
uni.getNetworkType().then(({ networkType }) => {
  if (networkType === 'none') {
    uni.showToast({ title: '网络已断开', icon: 'none' })
  }
})
```

### 离线数据暂存

```ts
// 提交失败时暂存到本地
const handleSubmit = async () => {
  try {
    await http({ url: '/api/add', data: formData })
    uni.showToast({ title: '提交成功' })
  } catch {
    // 网络异常，暂存离线
    const offlineList = uni.getStorageSync('offline_submit') || []
    offlineList.push({ url: '/api/add', data: formData, time: Date.now() })
    uni.setStorageSync('offline_submit', offlineList)
    uni.showToast({ title: '已暂存，网络恢复后自动提交', icon: 'none' })
  }
}

// 网络恢复时批量提交
const syncOfflineData = async () => {
  const offlineList = uni.getStorageSync('offline_submit') || []
  if (!offlineList.length) return

  for (const item of offlineList) {
    try {
      await http({ url: item.url, data: item.data })
      offlineList.splice(offlineList.indexOf(item), 1)
    } catch { break }
  }
  uni.setStorageSync('offline_submit', offlineList)
}
```

---

## 敏感数据安全

### Token 存储

```ts
// Token 通过 Pinia persist 存储，不要额外存一份
// persist 中配置 pick: ['token'] 即可

// 退出登录时清除
appStore.logout()  // 清空 store
uni.removeStorageSync('app')  // 清空持久化数据
uni.clearStorageSync()        // 清空所有（谨慎）
```

### 加密存储（可选）

```ts
import CryptoJS from 'crypto-js'

const SECRET_KEY = 'app-secret-key'

export const secureStorage = {
  set(key: string, value: any) {
    const encrypted = CryptoJS.AES.encrypt(JSON.stringify(value), SECRET_KEY).toString()
    uni.setStorageSync(key, encrypted)
  },
  get(key: string) {
    const encrypted = uni.getStorageSync(key)
    if (!encrypted) return null
    const decrypted = CryptoJS.AES.decrypt(encrypted, SECRET_KEY).toString(CryptoJS.enc.Utf8)
    return JSON.parse(decrypted)
  },
}
```
