# 条件编译规范

uni-app 使用条件编译实现一套代码多端运行。注意 `#ifdef` / `#ifndef` 是 uni-app 的**专有注释语法**，不是 JavaScript 的 `if` 语句。

## 基本语法

| 指令 | 含义 |
|------|------|
| `#ifdef PLATFORM` | 仅在指定平台编译 |
| `#ifndef PLATFORM` | 仅在非指定平台编译 |
| `#endif` | 结束条件块 |

## 平台标识

| 标识 | 平台 |
|------|------|
| `APP-PLUS` | App（Android/iOS） |
| `APP-PLUS-NVUE` | App nvue 页面 |
| `H5` | 移动端网页 |
| `MP-WEIXIN` | 微信小程序 |
| `MP-ALIPAY` | 支付宝小程序 |
| `MP-BAIDU` | 百度小程序 |
| `MP` | 所有小程序（微信/支付宝/百度/字节/QQ） |

---

## 在 template 中使用

```vue
<template>
  <view>
    <!-- App 专用 -->
    <!-- #ifdef APP-PLUS -->
    <view class="app-only">App 专属功能</view>
    <!-- #endif -->

    <!-- 非 App -->
    <!-- #ifndef APP-PLUS -->
    <view class="no-app">非 App 环境</view>
    <!-- #endif -->

    <!-- 小程序通用 -->
    <!-- #ifdef MP -->
    <official-account />
    <!-- #endif -->
  </view>
</template>
```

---

## 在 script 中使用

```ts
// #ifdef H5
const BASE_URL = '/api'
// #endif

// #ifdef APP-PLUS
const BASE_URL = 'https://api.example.com'
// #endif

// 多条件
// #ifdef MP-WEIXIN || MP-ALIPAY
console.log('小程序环境')
// #endif
```

---

## 在 style 中使用

```scss
/* #ifdef APP-PLUS */
.page {
  padding-top: var(--status-bar-height);  /* App 安全区域 */
}
/* #endif */

/* #ifdef H5 */
.page {
  padding-top: 0;
}
/* #endif */
```

---

## 在 pages.json 中使用

```json
{
  "pages": [
    {
      "path": "pages/index/index"
    },
    // #ifdef MP-WEIXIN
    {
      "path": "pages/wx-only/wx-only"
    }
    // #endif
  ]
}
```

---

## 常见场景

### 钉钉 JSAPI（仅 APP-PLUS）

```ts
// #ifdef APP-PLUS
import dd from 'dingtalk-jsapi'
// #endif

const dingLogin = () => {
  // #ifdef APP-PLUS
  dd.ready(() => { /* ... */ })
  // #endif
}
```

### 文件选择（平台差异）

```ts
const chooseFile = () => {
  // #ifdef H5
  // HTML input[type=file]
  // #endif

  // #ifdef APP-PLUS
  uni.chooseImage({ count: 1 })
  // #endif

  // #ifdef MP
  uni.chooseMessageFile({ count: 1 })
  // #endif
}
```

### API 地址（环境差异）

```ts
const getApiBase = () => {
  // #ifdef H5
  return '/api'
  // #endif

  // #ifdef APP-PLUS
  return 'https://api.example.com'
  // #endif
}
```

---

## 注意事项

1. 条件编译指令**必须独占一行**，不能和其他代码写在同一行
2. `#ifdef` / `#ifndef` 可以嵌套使用
3. 不要用条件编译包裹整个文件，会影响编辑器语法高亮
4. `vue.config.js`、`manifest.json` 中也可使用条件编译
