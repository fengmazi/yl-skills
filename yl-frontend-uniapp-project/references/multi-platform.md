# 多端差异处理

引领 uni-app 项目主要支持 3 个平台：APP-PLUS、H5、MP-ALIPAY（支付宝小程序）。

## 平台差异速查表

| 功能 | APP-PLUS | H5 | MP-ALIPAY | 处理方式 |
|------|:---:|:--:|:---:|------|
| 登录 | OAuth2 + 钉钉 JSAPI | OAuth2 | OAuth2 | 条件编译导入 dingtalk-jsapi |
| 文件上传 | uni.chooseImage | uni.chooseImage | uni.chooseImage | API 一致 |
| 文件下载 | uni.downloadFile | uni.downloadFile | uni.downloadFile | API 一致 |
| 推送 | 极光推送 | — | — | APP-PLUS 专用 |
| 版本更新 | plus.runtime 检测 | — | — | APP-PLUS 专用 |
| 权限 | plus.android 权限 | 浏览器权限 | 小程序权限配置 | platform-specific |
| CSS 单位 | rpx / px | rpx / px / vw/vh | rpx | H5 可额外使用 vw/vh |

---

## APP-PLUS 专有功能

### 版本更新检测

```ts
// #ifdef APP-PLUS
const checkUpdate = () => {
  plus.runtime.getProperty(plus.runtime.appid, (info) => {
    const currentVersion = info.version
    // 调后端接口获取最新版本号
    http({ url: '/system/version/latest' }).then(res => {
      if (res.version !== currentVersion) {
        uni.showModal({
          title: '发现新版本',
          content: '是否立即更新？',
          success: (result) => {
            if (result.confirm) {
              plus.runtime.openURL(res.downloadUrl)
            }
          },
        })
      }
    })
  })
}

// App.vue onLaunch 中调用
onLaunch(() => {
  checkUpdate()
})
// #endif
```

### 极光推送（车辆项目）

```ts
// #ifdef APP-PLUS
import JPush from '@/utils/jpush'

const initPush = () => {
  JPush.init()
  JPush.setAlias(appStore.userInfo.userId)
}

const logout = () => {
  JPush.deleteAlias()
  appStore.logout()
}
// #endif
```

---

## 路由栈管理

uni-app 的路由栈有层级限制（通常 10 层），超出会导致跳转失败：

```ts
const safeNavigate = (url: string) => {
  const pages = getCurrentPages()
  if (pages.length >= 9) {
    // 接近栈上限，使用 redirectTo 代替 navigateTo
    uni.redirectTo({ url })
  } else {
    uni.navigateTo({ url })
  }
}
```

---

## 安全区域适配

iPhone X 等全面屏设备的刘海/底部指示条适配：

```scss
.page {
  /* APP-PLUS 自动注入 --status-bar-height 和 --safe-area-inset-bottom */
  padding-top: var(--status-bar-height, 0);
  padding-bottom: var(--safe-area-inset-bottom, 0);
}
```

---

## TabBar 注意事项

| 规则 | 说明 |
|------|------|
| 最少 2 个、最多 5 个 tab | uni-app 限制 |
| pagePath 不能以 `/` 开头 | 微信小程序兼容 |
| tab 跳转必须用 `uni.switchTab` | 不能用 `uni.navigateTo` |
| tab 页面不能带参数 | 需要通过 store 或全局变量传参 |

---

## 支付宝小程序特殊处理

### 高德 API 密钥

```json
// manifest.json
{
  "mp-alipay": {
    "appid": "xxx",
    "setting": {
      "urlCheck": false
    }
  }
}
```

### 支付

支付宝小程序使用 `uni.requestPayment` + `provider: 'alipay'`：

```ts
uni.requestPayment({
  provider: 'alipay',
  orderInfo: 'xxx',  // 后端返回的订单字符串
  success: () => {
    uni.showToast({ title: '支付成功' })
  },
})
```
