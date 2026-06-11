# 构建与分包优化

## 构建命令

```bash
# 开发
pnpm dev:app          # App 端
pnpm dev:h5           # H5 端
pnpm dev:mp-alipay    # 支付宝小程序
pnpm dev:mp-weixin    # 微信小程序

# 打包
pnpm build:app        # App 端
pnpm build:h5         # H5 端
pnpm build:mp-alipay  # 支付宝小程序
pnpm build:mp-weixin  # 微信小程序
```

---

## pages.json 分包配置

### 为什么需要分包

小程序主包大小有限制：
- 微信小程序：主包 ≤ 2MB，总包 ≤ 20MB
- 支付宝小程序：主包 ≤ 2MB，总包 ≤ 8MB

### 分包规则

| 规则 | 说明 |
|------|------|
| tabBar 页面必须在主包 | `switchTab` 的目标页面不能分包 |
| 分包不能互相引用 | 分包 A 不能 import 分包 B 的组件 |
| 公共组件放主包 | 多个分包共用的组件放在 `components/` |
| 静态资源分包 | 大图片、字体等跟随分包存放 |

### 配置示例

```json
{
  "pages": [
    { "path": "pages/index/index" },
    { "path": "pages/user/user" }
  ],
  "subPackages": [
    {
      "root": "pages/task",
      "pages": [
        { "path": "list" },
        { "path": "detail" },
        { "path": "apply" }
      ]
    },
    {
      "root": "pages/vehicle",
      "pages": [
        { "path": "list" },
        { "path": "dispatch" },
        { "path": "track" }
      ]
    }
  ],
  "preloadRule": {
    "pages/index/index": {
      "network": "all",
      "packages": ["pages/task"]
    }
  }
}
```

### 预加载策略

```json
{
  "preloadRule": {
    "pages/index/index": {
      "network": "all",       // 所有网络都预加载
      "packages": ["pages/task"]
    },
    "pages/task/list": {
      "network": "wifi",      // 仅 WiFi 下预加载
      "packages": ["pages/vehicle"]
    }
  }
}
```

| network 值 | 说明 |
|------------|------|
| all | 任何网络都预加载 |
| wifi | 仅 WiFi 预加载 |

---

## 静态资源优化

### 图片规范

| 类型 | 建议 |
|------|------|
| 图标 | 用 `uni-icons` 或 iconfont，不用图片 |
| 背景图 | 优先用 CSS 渐变，其次 base64 小图 |
| 大图 | 放在对应分包目录，避免在主包 |
| 云存储 | 超 50KB 的图片上传到 OSS，前端引用 URL |

### 字体文件

```css
/* 避免引入完整字体文件（动辄几 MB） */
/* 方案 1：子集化（只包含需要的字） */
@font-face {
  font-family: 'CustomFont';
  src: url('~@/static/font/custom-subset.ttf');
}

/* 方案 2：放在对应分包 */
/* 方案 3：上传 OSS，用 URL 引用 */
```

---

## 打包产物分析

### 查看包体积

```bash
# 小程序打包后查看各分包大小
# 产物在 dist/build/mp-alipay/ 下
ls -lh dist/build/mp-alipay/

# 查看具体文件
du -sh dist/build/mp-alipay/*/
```

### 常见大文件来源

| 来源 | 解决方法 |
|------|----------|
| `echarts` + `zrender` | 条件编译排除小程序平台 |
| `moment.js` | 换 `dayjs`（2KB vs 70KB） |
| 字体文件 | 子集化或 OSS |
| 大图片 | OSS |
| 未 tree-shaking 的依赖 | 检查 import 是否引入了整个库 |

---

## App 端打包

### Android

```bash
# 本地打包
pnpm build:app
# 产物在 dist/build/app/

# 使用 HBuilderX 云打包或本地离线打包
```

### iOS

需要 macOS + Xcode，通过 HBuilderX 云打包或本地打包。

### App 端特有配置

```json
// manifest.json
{
  "app-plus": {
    "distribute": {
      "android": {
        "minSdkVersion": 19,
        "abiFilters": ["armeabi-v7a", "arm64-v8a"]
      },
      "ios": {
        "dSYMs": false
      },
      "sdkConfigs": {
        "push": {
          "jpush": {
            "appkey_android": "xxx",
            "channel_android": "default"
          }
        }
      }
    }
  }
}
```
