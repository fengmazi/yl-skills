# uni-app 启动项配置

## package.json Scripts

```json
{
  "scripts": {
    "dev:app": "uni -p app",
    "dev:h5": "uni",
    "dev:mp-alipay": "uni -p mp-alipay",
    "dev:mp-weixin": "uni -p mp-weixin",
    "build:app": "uni build -p app",
    "build:h5": "uni build",
    "build:mp-alipay": "uni build -p mp-alipay",
    "build:mp-weixin": "uni build -p mp-weixin"
  }
}
```

---

## manifest.json 关键配置

```json
{
  "name": "应用名称",
  "appid": "__UNI__XXXXXX",
  "versionName": "1.0.0",
  "versionCode": "100",

  "mp-alipay": {
    "appid": "支付宝小程序AppID",
    "setting": {
      "urlCheck": false
    }
  },

  "mp-weixin": {
    "appid": "微信小程序AppID",
    "setting": {
      "urlCheck": false
    }
  },

  "app-plus": {
    "distribute": {
      "android": {
        "minSdkVersion": 19
      }
    }
  }
}
```

---

## pages.json 关键配置

### tabBar

```json
{
  "tabBar": {
    "color": "#999999",
    "selectedColor": "#007AFF",
    "backgroundColor": "#FFFFFF",
    "borderStyle": "black",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "static/tab/home.png",
        "selectedIconPath": "static/tab/home-active.png"
      },
      {
        "pagePath": "pages/project/project",
        "text": "项目"
      },
      {
        "pagePath": "pages/user/user",
        "text": "我的"
      }
    ]
  }
}
```

> 注意：`tabBar.list` 的 `pagePath` 开头**不要**带 `/`（与微信小程序兼容）。

### subPackages（分包）

当页面数较多时使用分包优化首屏加载：

```json
{
  "subPackages": [
    {
      "root": "pages/task",
      "pages": [
        { "path": "list" },
        { "path": "detail" }
      ]
    }
  ]
}
```

### 全局样式

```json
{
  "globalStyle": {
    "navigationBarTextStyle": "white",
    "navigationBarTitleText": "应用名称",
    "navigationBarBackgroundColor": "#007AFF",
    "backgroundColor": "#F5F5F5"
  }
}
```

---

## 环境变量与条件编译

### .env 文件（部分项目使用）

uni-app 通过 `VITE_APP_*` 前缀（Vue 3 / Vite 模式）注入环境变量：

```env
# .env.development
VITE_APP_BASE_API = '/api'
VITE_APP_DINGDING_APPID = 'xxx'

# .env.production
VITE_APP_BASE_API = 'https://api.example.com'
```

### 条件编译区分环境

```ts
const BASE_URL = import.meta.env.VITE_APP_BASE_API
```

---

## 项目依赖清单

### 核心依赖

```json
{
  "@dcloudio/uni-app": "3.0.0-xxx",
  "vue": "^3.3.11",
  "pinia": "^2.1.7",
  "pinia-plugin-persistedstate": "^3.2.0",
  "dayjs": "^1.11.13",
  "dingtalk-jsapi": "^3.1.0"
}
```

### 可选（按项目需要）

```json
{
  "echarts": "^5.4.3 / ^6.0.0",
  "zrender": "^6.0.0",
  "js-base64": "^3.7.8",
  "jsencrypt": "^3.3.2"
}
```
