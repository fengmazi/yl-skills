# Vue 2 启动项配置

## vue.config.js 模板

Vue CLI 3 项目通过 `vue.config.js` 配置 webpack 和 devServer：

```js
const port = process.env.VUE_APP_PORT || 3012
const proxyTarget = process.env.VUE_APP_PROXY_TARGET || 'http://192.168.1.2:8013/DTCB'
const proxyPath = process.env.VUE_APP_BASE_URL || '/api'

module.exports = {
  publicPath: '/',
  outputDir: 'dist',
  productionSourceMap: false,  // 生产不生成 source map

  devServer: {
    port: Number(port),
    proxy: {
      [proxyPath]: {
        target: proxyTarget,
        changeOrigin: true,
        pathRewrite: {
          ['^' + proxyPath]: '',  // Vue CLI 必须写 pathRewrite
        },
      },
    },
  },

  chainWebpack: config => {
    // snap.svg 特殊处理
    config.module
      .rule('snapsvg')
      .test(require.resolve('snapsvg'))
      .use('imports-loader')
      .loader('imports-loader?this=>window,fix=>module.exports=0')
      .end()
  },
}
```

### 关键点

- Vue CLI 通过 `process.env` 读取 `.env.*` 文件中的 `VUE_APP_*` 变量
- 代理**必须写 `pathRewrite`** 去掉前缀（后端路径前缀与前端 axios baseURL 不同）
- `--mode` 参数决定加载哪个 `.env.[mode]` 文件
- 输出目录固定为 `dist/`
- 生产环境不生成 source map（`productionSourceMap: false`）

---

## .env 文件模板

### .env.development（默认开发）

```env
VUE_APP_BASE_URL = '/api'
VUE_APP_PROXY_TARGET = 'http://192.168.1.2:8013/DTCB'
VUE_APP_PORT = 3012
VUE_APP_ENV = 'dev'
```

### .env.dev-onlineTest（测试联调）

```env
VUE_APP_BASE_URL = '/api'
VUE_APP_PROXY_TARGET = 'http://dtcb-ytq.erp12580.com/DTCB'
VUE_APP_PORT = 9012
VUE_APP_ENV = 'dev'
```

### .env.prod（生产打包）

```env
NODE_ENV = 'production'
VUE_APP_BASE_URL = '/DTCB'
VUE_APP_OSS_PATH = 'https://dtcb.ycsyysnh.com:5001/'
VUE_APP_ENV = 'prod'
VUE_APP_DINGDING_APPID = '正式appid'
VUE_APP_ENABLE_BAIDU_TONGJI = 'tongji-id'
```

---

## Scripts 模板

```json
{
  "scripts": {
    "dev": "vue-cli-service serve",
    "dev:local": "vue-cli-service serve --mode dev-local",
    "dev:onlineTest": "vue-cli-service serve --mode dev-onlineTest",
    "build": "vue-cli-service build",
    "build-prod": "vue-cli-service build --mode prod && node zip.cjs",
    "deploy": "npm run build && node ./deploy.js"
  }
}
```

---

## package.json 关键依赖

### 运行时

```json
{
  "vue": "^2.6.10",
  "element-ui": "^2.15.14",
  "vue-router": "^3.0.3",
  "vuex": "^3.0.1",
  "axios": "^1.7.2",
  "echarts": "^5.6.0",
  "vue2-datepicker": "^3.11.1",
  "vuex-persistedstate": "^2.5.4",
  "noty": "^3.2.0-beta"
}
```

### 构建

```json
{
  "@vue/cli-service": "^3.9.0",
  "@vue/cli-plugin-babel": "^3.9.0",
  "vue-template-compiler": "^2.6.10",
  "sass": "^1.32.12",
  "sass-loader": "^7.1.0"
}
```

### 部署

```json
{
  "node-scp": "^1.x",
  "ssh2": "^1.x",
  "chalk": "^5.x",
  "ora": "^8.x"
}
```

---

## 部署相关内容

部署脚本（deploy.js、zip.cjs、serverConfig.mjs）详见 `yl-frontend-pc-project-shared` skill 的 `references/deployment.md`。
