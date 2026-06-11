# uni-app 架构

## 目录布局

uni-app 项目**没有传统的 `src/` 目录**，源码直接在项目根目录，`pages/` 作为页面入口。

```
项目根目录/
├── App.vue              # 应用入口
├── main.ts              # 应用初始化
├── pages.json           # 页面路由 + 全局配置
├── manifest.json        # 应用配置
├── pages/               # 页面
├── components/          # 组件
├── stores/              # Pinia
├── utils/               # 工具
├── enums/               # 枚举
├── hooks/               # hooks
└── static/              # 静态资源
```

---

## 页面组织方式

4 个项目采用了 3 种不同方式：

| 方式 | 示例 | 项目 |
|------|------|------|
| **目录分层** | `pages/task/task.vue`, `pages/vehicle/apply.vue` | budget-app（31 页） |
| **扁平 .vue** | `pages/login.vue`, `pages/index.vue` | budget-bt-app（15 页）、vehicle-app-new（20 页） |
| **混合模式** | 既有 `.vue` 也有目录 | costfeecontrol-app（41 页） |

> 新增页面时，参考当前项目的既有组织方式保持一致。

---

## App.vue — 应用入口

所有项目的 `App.vue` 使用 **Options API**（处理 uni-app 生命周期），而非 `<script setup>`：

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  onLaunch() {
    // 应用启动：检查登录状态、获取系统信息
  },
  onShow() {
    // 应用显示
  },
  onHide() {
    // 应用隐藏
  },
})
</script>
```

> 原因：`onLaunch`、`onShow`、`onHide` 等 uni-app 生命周期钩子在 Options API 中表达更自然。

---

## main.ts — 应用初始化

```ts
import { createSSRApp } from 'vue'
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
import App from './App.vue'

export function createApp() {
  const app = createSSRApp(App)
  const pinia = createPinia()
  pinia.use(piniaPluginPersistedstate)
  app.use(pinia)
  return { app, pinia }
}
```

关键点：
- 使用 `createSSRApp`（uni-app 的 SSR 兼容入口）
- Pinia 持久化插件配置 `storage: uni.getStorageSync` 适配 uni-app 存储

---

## pages.json — 页面路由

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
        "navigationBarTitleText": "首页"
      }
    }
  ],
  "subPackages": [],
  "tabBar": {
    "list": [
      { "pagePath": "pages/index/index", "text": "首页" },
      { "pagePath": "pages/user/user", "text": "我的" }
    ]
  },
  "globalStyle": {
    "navigationBarTextStyle": "white",
    "navigationBarTitleText": "应用名称",
    "navigationBarBackgroundColor": "#007AFF"
  }
}
```

**注意**：`tabBar.list` 中 `pagePath` 开头**不要**带 `/`（与微信小程序规范一致）。

---

## manifest.json — 应用配置

```json
{
  "name": "应用名称",
  "appid": "__UNI__XXXXXX",
  "versionName": "1.0.0",
  "mp-alipay": {
    "appid": "支付宝小程序AppID"
  }
}
```

---

## 平台支持矩阵

| 功能 | APP-PLUS | H5 | MP-ALIPAY | MP-WEIXIN |
|------|:---:|:--:|:---:|:---:|
| 基础页面 | ✓ | ✓ | ✓ | ✓ |
| 钉钉登录 | ✓ | ✓ | ✗ | ✗ |
| 文件上传 | ✓ | ✓ | ✓ | ✓ |
| 极光推送 | ✓ | ✗ | ✗ | ✗ |
| 版本更新 | ✓ | ✗ | ✗ | ✗ |
| ECharts | ✓ | ✓ | ✗ | ✗ |
