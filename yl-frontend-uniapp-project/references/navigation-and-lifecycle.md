# 导航与生命周期

## App 生命周期

`App.vue` 中使用 Options API 处理 uni-app 生命周期：

```vue
<script lang="ts">
export default {
  onLaunch(options) {
    // 应用初始化：检查登录、获取系统信息
    // options.path — 启动路径
    // options.query — 启动参数
    // options.scene — 场景值（小程序）
  },
  onShow(options) {
    // 应用从后台进入前台
  },
  onHide() {
    // 应用进入后台
  },
  onError(err) {
    // 应用异常捕获
    console.error('App Error:', err)
  },
  onUnhandledRejection({ reason, promise }) {
    // 未处理的 Promise reject
  },
  onPageNotFound({ path, query, isEntryPage }) {
    // 页面不存在，可动态重定向
    uni.redirectTo({ url: '/pages/404/404' })
  },
}
</script>
```

---

## 页面生命周期

```vue
<script setup lang="ts">
import { onLoad, onShow, onReady, onHide, onUnload } from '@dcloudio/uni-app'

// 页面加载（只触发一次，可获取路由参数）
onLoad((options) => {
  console.log('页面参数:', options?.id, options?.type)
})

// 页面显示（每次进入都触发）
onShow(() => {
  // 刷新数据
})

// 页面初次渲染完成
onReady(() => {
  // 获取节点信息等
})

// 页面隐藏
onHide(() => {
  // 暂停音视频、清除定时器等
})

// 页面卸载
onUnload(() => {
  // 销毁实例、取消请求等
  clearTaskList()
})

// 下拉刷新
onPullDownRefresh(async () => {
  await loadData()
  uni.stopPullDownRefresh()
})

// 触底加载更多
onReachBottom(() => {
  if (!finished.value) loadMore()
})
</script>
```

### 生命周期触发顺序

```
App.onLaunch → Page.onLoad → Page.onShow → Page.onReady
                                         → Component.mounted
```

切换 tab 页：
```
Page.onShow（不会触发 onLoad）
```

返回上一页：
```
当前页 onUnload → 上一页 onShow
```

---

## 路由导航

### 导航 API

| API | 说明 | 栈变化 |
|-----|------|--------|
| `uni.navigateTo` | 打开新页面 | 压栈 +1 |
| `uni.redirectTo` | 替换当前页 | 栈不变 |
| `uni.reLaunch` | 关闭所有页面，打开新页 | 重置栈 |
| `uni.switchTab` | 切换到 tab 页 | 重置非 tab 栈 |
| `uni.navigateBack` | 返回上一页 | 出栈 -1 |

### 参数传递

```ts
// 方式 1：URL query（最常用）
uni.navigateTo({ url: `/pages/detail/detail?id=123&type=A` })

// onLoad 接收
onLoad((options) => {
  const id = options?.id      // "123"
  const type = options?.type  // "A"
})
```

```ts
// 方式 2：全局 eventBus（复杂对象）
// 发送方
uni.$emit('dataChange', { id: 123, name: 'xxx' })

// 接收方
uni.$on('dataChange', (data) => {
  console.log(data)
})

// 页面卸载时取消监听
onUnload(() => {
  uni.$off('dataChange')
})
```

```ts
// 方式 3：Pinia store（跨多页共享）
const appStore = useAppStore()
appStore.setPageData({ id: 123 })
```

### 返回传参

```ts
// 页面 A → 页面 B → 返回 A 时传数据

// 页面 B 中
const pages = getCurrentPages()
const prevPage = pages[pages.length - 2]
prevPage.$vm.handleResult(data)  // 调用上一页的方法
uni.navigateBack()

// 页面 A 中暴露方法
const handleResult = (data) => {
  console.log('B 返回的数据:', data)
}
```

---

## 页面栈管理

```ts
// 获取完整页面栈
const pages = getCurrentPages()
// pages[0] — 首页
// pages[pages.length - 1] — 当前页

// 跳转到页面栈中的某个页面
uni.navigateBack({ delta: pages.length - 2 })  // 回到第二个页面

// 判断是否从某个页面来的
onShow(() => {
  const pages = getCurrentPages()
  const fromPage = pages[pages.length - 2]
  if (fromPage?.route === 'pages/login/login') {
    // 从登录页返回
    loadData()
  }
})
```

---

## 常见路由场景

### 登录跳转

```ts
// 需要登录的页面
onLoad(() => {
  if (!appStore.token) {
    uni.reLaunch({ url: '/pages/login/login' })
    return
  }
  loadData()
})

// 登录成功后回到目标页
const handleLogin = async () => {
  await doLogin()
  const redirectUrl = appStore.redirectUrl || '/pages/index/index'
  uni.reLaunch({ url: redirectUrl })
}
```

### 表单提交后返回

```ts
const handleSubmit = async () => {
  await http({ url: '/api/add', data: formData })
  uni.showToast({ title: '提交成功' })
  // 延迟返回，让用户看到提示
  setTimeout(() => {
    uni.navigateBack()
  }, 1500)
}
```

### 深层链接处理

```ts
// App.vue onLaunch
onLaunch((options) => {
  // 从推送、分享链接进入时
  if (options.path) {
    appStore.setDeepLink(options.path, options.query)
  }
})

// pages/index/index onShow
onShow(() => {
  const deepLink = appStore.deepLink
  if (deepLink) {
    uni.navigateTo({ url: deepLink })
    appStore.clearDeepLink()
  }
})
```
