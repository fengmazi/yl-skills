# uni-app 常见陷阱与调试

## 路由栈溢出

uni-app 路由栈默认最多 5 层（微信小程序）/ 10 层（App），超出导致 `navigateTo` 失败（页面不跳转但也不报错）。

### 解决方案

```ts
// 安全导航：接近栈上限时自动降级为 redirectTo
const safeNavigate = (url: string) => {
  const pages = getCurrentPages()
  if (pages.length >= 9) {
    uni.redirectTo({ url })
  } else {
    uni.navigateTo({ url })
  }
}
```

### 排查方法

```ts
// 查看当前路由栈深度
console.log(getCurrentPages().length)
console.log(getCurrentPages().map(p => p.route))
```

---

## TabBar 导航陷阱

| 陷阱 | 说明 |
|------|------|
| tab 页面只能用 `switchTab` | `navigateTo` 到 tab 页会静默失败 |
| `switchTab` 不能传参 | 参数需通过 store 或全局变量 |
| tab 页面无法带 query | `?id=123` 在 tab 页上无效 |
| tab 页面 `onLoad` 只触发一次 | 切换 tab 时触发 `onShow`，不是 `onLoad` |

### 变通传参

```ts
// 发送方
appStore.setTempData({ id: 123 })
uni.switchTab({ url: '/pages/index/index' })

// 接收方（index.vue）
onShow(() => {
  const data = appStore.tempData
  if (data?.id) {
    loadDetail(data.id)
    appStore.clearTempData()
  }
})
```

---

## 条件编译常见错误

### 不要在 JS 表达式中混用

```ts
// 错误 — 条件编译只能在行级使用
const url = process.env.NODE_ENV === 'development' ? '/dev' : '/prod'

// 正确
// #ifdef H5
const url = '/dev'
// #endif
// #ifdef APP-PLUS
const url = '/prod'
// #endif
```

### 样式中的条件编译

```scss
/* #ifdef APP-PLUS */
.status-bar { padding-top: var(--status-bar-height); }
/* #endif */

/* #ifndef APP-PLUS */
.status-bar { padding-top: 0; }
/* #endif */
```

---

## 组件通信陷阱

### 跨层级通信

uni-app 中 Vue 3 的 `provide/inject` 在部分平台有兼容问题，跨页面传递数据优先用 Pinia store：

```ts
// 正确 — 跨页面用 Store
appStore.setFormData({ name: 'xxx' })

// 避免 — 跨页面 provide/inject 在 App 端可能失效
provide('formData', formData)
```

### $refs 在 uni-app 中

```vue
<template>
  <my-form ref="formRef" />
</template>

<script setup>
// uni-app 中 ref 获取组件实例
const formRef = ref()
formRef.value?.validate()  // 需要组件暴露方法

// 子组件中
defineExpose({ validate })
</script>
```

---

## 性能陷阱

### 长列表性能

普通 `v-for` 在数据超过 200 条时滚动可能卡顿：

```vue
<!-- 使用虚拟列表组件 -->
<recycle-view :list="list">
  <recycle-item v-for="item in list" :key="item.id">
    <!-- 每一项内容 -->
  </recycle-item>
</recycle-view>
```

### setData 频率

`uni-app` 底层通过 `setData` 通信，频繁更新大数据对象会导致卡顿：

```ts
// 避免 — 频繁全量更新
setInterval(() => {
  list.value = newLargeList
}, 1000)

// 正确 — 只更新变化的项
list.value[index] = newItem
```

### 图片优化

```html
<!-- 懒加载 -->
<image :src="url" lazy-load mode="aspectFill" />

<!-- 指定宽高避免重排 -->
<image :src="url" style="width: 200rpx; height: 200rpx" />
```

---

## 样式陷阱

### 不支持的选择器

uni-app 在一些平台上不支持：
- `*` 通配符
- 属性选择器 `[attr]`
- 兄弟选择器 `+`、`~`

### rpx 转换注意

- `1rpx` = 屏幕宽度 / 750
- App/H5 端支持 `vw/vh`，小程序不支持
- `750rpx` ≈ 100vw ≈ 屏幕宽度
- 字体大小建议用 `rpx` 而非 `px`（不同屏幕尺寸表现一致）

### 滚动穿透

弹窗内滚动穿透到背景页：

```scss
// 弹窗打开时锁定背景
.page-locked {
  position: fixed;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
```

---

## 请求陷阱

### 请求超时

默认超时可能太短（部分平台默认 30s，部分 60s），大数据查询应单独设置：

```ts
http({ url: '/api/report', data: params, timeout: 90000 })
```

### 并发请求限制

微信小程序限制 `wx.request` 并发数（默认 10 个），超出会导致请求排队/失败：

```ts
// 大量并发请求时用队列控制
const MAX_CONCURRENT = 5
```

---

## 调试技巧

### 真机调试

```ts
// App 端打开调试
// #ifdef APP-PLUS
console.log('当前页面:', getCurrentPages().map(p => p.route))
// #endif
```

### vconsole（H5）

```html
<!-- #ifdef H5 -->
<script src="https://unpkg.com/vconsole@latest/dist/vconsole.min.js"></script>
<script>new VConsole()</script>
<!-- #endif -->
```

### 小程序调试

支付宝小程序开发工具中的 `my` API 调试面板可查看所有 `uni.*` 调用的底层发送数据。
