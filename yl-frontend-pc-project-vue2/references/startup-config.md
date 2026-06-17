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

部署脚本（deploy.js、serverConfig.mjs）详见 `yl-frontend-pc-project-shared` skill 的 `references/deployment.md`，`zip.cjs` 模板见下文。

## zip.cjs（构建后打包）

`build-prod` 构建完成后自动调用 `node zip.cjs`，将 `dist/` 打包为 `dist.zip`，完成后打开文件管理器定位到该文件。

### Scripts 集成

```json
{
  "scripts": {
    "build-prod": "vue-cli-service build --mode prod && node zip.cjs"
  }
}
```

### 跨平台压缩策略

按优先级依次尝试，原生命令优先，Bandizip 作为兜底：

| 优先级 | Windows | macOS | Linux |
|--------|---------|-------|-------|
| 1（原生） | `tar -a -cf dist.zip dist` | `zip -r dist.zip dist/` | `zip -r dist.zip dist/` |
| 2（兜底） | `bz.exe c -r dist.zip dist` | `bz c -r dist.zip dist/` | `bz c -r dist.zip dist/` |

> Windows 原生 tar 从 Win 10 1803 起内置支持 `-a` 参数生成 zip。Bandizip 命令行需将 `bz.exe` 所在目录加入 PATH。

### 完整模板

```javascript
const { exec } = require('child_process')
const path = require('path')
const os = require('os')

function getOSType() {
  const platform = os.platform()
  if (platform === 'win32') return 'windows'
  if (platform === 'darwin') return 'mac'
  return 'linux'
}

function getZipCommands() {
  const osType = getOSType()
  switch (osType) {
    case 'windows':
      return [
        { cmd: 'tar -a -cf dist.zip dist', name: 'Windows 原生 tar' },
        { cmd: 'bz.exe c -r dist.zip dist', name: 'Bandizip (bz.exe)' }
      ]
    case 'mac':
      return [
        { cmd: 'zip -r dist.zip dist/', name: 'macOS 原生 zip' },
        { cmd: 'bz c -r dist.zip dist/', name: 'Bandizip (bz)' }
      ]
    case 'linux':
      return [
        { cmd: 'zip -r dist.zip dist/', name: 'Linux 原生 zip' },
        { cmd: 'bz c -r dist.zip dist/', name: 'Bandizip (bz)' }
      ]
    default:
      throw new Error(`不支持的操作系统: ${osType}`)
  }
}

function getOpenCommand(filePath) {
  const osType = getOSType()
  switch (osType) {
    case 'windows':
      return `start "" explorer /select,"${filePath}"`
    case 'mac':
      return `open -R "${filePath}"`
    case 'linux':
      return `xdg-open "${path.dirname(filePath)}"`
    default:
      throw new Error(`不支持的操作系统: ${osType}`)
  }
}

function execPromise(command) {
  return new Promise((resolve, reject) => {
    exec(command, { maxBuffer: 10 * 1024 * 1024 }, (error, stdout, stderr) => {
      if (error) reject({ error, stderr: stderr.trim() })
      else resolve(stdout)
    })
  })
}

async function tryZipCommands(commands) {
  const errors = []
  for (const { cmd, name } of commands) {
    try {
      console.log(`正在尝试 ${name} ...`)
      await execPromise(cmd)
      console.log(`✓ ${name} 压缩成功`)
      return
    } catch (err) {
      console.log(`✗ ${name} 失败: ${err.error.message}`)
      errors.push({ name, message: err.error.message, stderr: err.stderr })
    }
  }
  const errorMsg = errors.map(e => {
    let detail = `  · ${e.name}: ${e.message}`
    if (e.stderr) detail += `\n    错误详情: ${e.stderr}`
    return detail
  }).join('\n')
  throw new Error(`所有压缩方式均失败，请检查环境配置：\n${errorMsg}`)
}

const file = path.resolve(__dirname, 'dist.zip')
const commands = getZipCommands()

tryZipCommands(commands)
  .then(() => {
    const openCommand = getOpenCommand(file)
    return execPromise(openCommand)
  })
  .catch(error => {
    console.error('\n打包失败:', error.message)
    process.exit(1)
  })
```

### 依赖要求

| 依赖 | 必需 | 说明 |
|------|------|------|
| Node.js | 是 | 运行脚本的运行时 |
| 系统 tar/zip | 否 | 各系统自带，优先使用 |
| Bandizip | 否 | 仅当原生命令不可用时作为 fallback |
