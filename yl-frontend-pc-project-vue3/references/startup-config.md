# 启动项配置模板

## Scripts 模板

### Vite 项目（Vue 3）

```json
{
  "scripts": {
    "dev": "vite --host",
    "dev:local": "vite --host --mode dev-local",
    "dev:onlineTest": "vite --host --mode dev-onlineTest",
    "build": "vue-tsc && vite build",
    "build-prod": "vue-tsc --noEmit && vite build --mode prod && node zip.cjs",
    "preview": "vite preview",
    "deploy": "pnpm build && node ./deploy.js",
    "prepare": "husky install",
    "commit": "cz"
  }
}
```

### Vue CLI 项目（Vue 2 / Webpack）

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

## .env 文件模板

### .env.dev-local（本地开发，连本地后端电脑，端口 30xx）

```env
# Vite 项目用 VITE_APP_* 前缀，Vue CLI 项目用 VUE_APP_* 前缀

ENV = 'development'
VUE_APP_ENV = 'dev'
VITE_APP_ENV = 'dev'

# base api（axios baseURL，也是代理匹配路径）
VUE_APP_BASE_URL = '/api'
VITE_APP_BASE_API = '/api'

# 代理目标 → 本地后端电脑
VUE_APP_PROXY_TARGET = 'http://192.168.1.2:8013/项目路径'
VITE_APP_PROXY_TARGET = 'http://192.168.1.2:8013'

# 端口（30 开头）
VUE_APP_PORT = 3012
VITE_APP_PORT = 3005

# OSS 路径
VUE_APP_OSS_PATH = 'http://oss.erp12580.com/'
VITE_APP_OSS_PATH = 'http://oss.erp12580.com/'

# 百度统计（开发环境通常关闭）
VUE_APP_ENABLE_BAIDU_TONGJI = ''

# 钉钉 AppID
VUE_APP_DINGDING_APPID = 'xxx'
VITE_APP_DINGDING_APPID = 'xxx'

# 公司信息（开发环境通常关闭）
VUE_APP_SHOW_INFO = 'N'
VITE_APP_SHOW_INFO = 'N'
```

### .env.dev-onlineTest（测试环境，端口 90xx）

```env
ENV = 'development'
VUE_APP_ENV = 'dev'
VITE_APP_ENV = 'dev'

VUE_APP_BASE_URL = '/api'
VITE_APP_BASE_API = '/api'

# 代理目标 → 测试环境
VUE_APP_PROXY_TARGET = 'http://项目名.erp12580.com/项目路径'
VITE_APP_PROXY_TARGET = 'http://项目名.erp12580.com'

# 端口（90 开头）
VUE_APP_PORT = 9012
VITE_APP_PORT = 9005

VUE_APP_OSS_PATH = 'http://oss.erp12580.com/'
VITE_APP_OSS_PATH = 'http://oss.erp12580.com/'
VUE_APP_ENABLE_BAIDU_TONGJI = ''
VUE_APP_DINGDING_APPID = 'xxx'
VITE_APP_DINGDING_APPID = 'xxx'
VUE_APP_SHOW_INFO = 'N'
VITE_APP_SHOW_INFO = 'N'
```

### .env.prod（生产环境打包，--mode prod）

```env
ENV = 'production'
NODE_ENV = 'production'
VUE_APP_ENV = 'prod'
VITE_APP_ENV = 'prod'

# base api（生产环境直接用后端路径，不走代理）
VUE_APP_BASE_URL = '/项目路径'
VITE_APP_BASE_API = '/项目路径'

# 生产 OSS 路径（通常不同于开发）
VUE_APP_OSS_PATH = 'https://项目域名.com:5001/'
VITE_APP_OSS_PATH = 'http://10.10.2.30:9000/'

# 百度统计（生产环境按需开启）
VUE_APP_ENABLE_BAIDU_TONGJI = ''

# 钉钉 AppID（生产环境用正式 appid）
VUE_APP_DINGDING_APPID = '正式appid'
VITE_APP_DINGDING_APPID = '正式appid'

# 公司信息（生产环境开启）
VUE_APP_SHOW_INFO = 'Y'
VITE_APP_SHOW_INFO = 'Y'
```

---

## vite.config.ts 模板

```ts
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import { fileURLToPath, URL } from 'url'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')

  return {
    server: {
      port: Number(env.VITE_APP_PORT) || 3005,
      proxy: {
        [env.VITE_APP_BASE_API]: {
          target: env.VITE_APP_PROXY_TARGET,
          changeOrigin: true,
        },
      },
    },
    resolve: {
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url)),
      },
    },
    plugins: [vue(), vueJsx()],
  }
})
```

关键点：
- `loadEnv(mode, ...)` 根据 `--mode` 加载对应的 `.env.[mode]` 文件
- `VITE_APP_BASE_API` 既做 axios baseURL 也做代理匹配路径
- Vite 代理不写 pathRewrite，默认透传完整路径

---

## vue.config.js 模板

```js
const port = process.env.VUE_APP_PORT || 3012
const proxyTarget = process.env.VUE_APP_PROXY_TARGET || 'http://192.168.1.2:8013/项目路径'
const proxyPath = process.env.VUE_APP_BASE_URL || '/api'

module.exports = {
  publicPath: '/',
  outputDir: 'dist',
  productionSourceMap: false,
  devServer: {
    port: Number(port),
    proxy: {
      [proxyPath]: {
        target: proxyTarget,
        changeOrigin: true,
        pathRewrite: {
          ['^' + proxyPath]: '',
        },
      },
    },
  },
}
```

关键点：
- Vue CLI 通过 `process.env` 读取 `.env.*` 文件中的 `VUE_APP_*` 变量
- Vue CLI 代理通常需要 `pathRewrite` 去掉前缀（因为后端路径前缀与前端 axios baseURL 不同）
- `--mode` 参数决定加载哪个 `.env.[mode]` 文件

---

## deploy.js（部署到测试服务器）

通过 SSH + SCP 部署 `dist/` 到测试服务器，包含备份、清理、上传、本地清理完整流程。

### 完整模板

```js
Promise.all([import('ora'), import('chalk'), import('node-scp'), import('ssh2'), import('fs'), import('./serverConfig.mjs')]).then(
  ([oraModule, chalkModule, scpClientModule, ClientModule, fsModule, serverConfigModule]) => {
    const ora = oraModule.default
    const chalk = chalkModule.default
    const scpClient = scpClientModule.default
    const Client = ClientModule.Client
    const serverConfig = serverConfigModule.default

    const conn = new Client()
    const connect = () => {
      return new Promise(resolve => {
        conn.on('ready', resolve).connect(serverConfig)
      })
    }
    const connExec = cmd => {
      console.log('excete -> ' + cmd)
      return new Promise((resolve, reject) => {
        conn.exec(cmd, (err, stream) => {
          if (err) reject(err)
          stream
            .on('close', () => {
              console.log('close -> ' + cmd)
              resolve()
            })
            .on('data', data => {
              data && resolve(data)
            })
            .stderr.on('data', data => {
              reject(data)
            })
        })
      })
    }

    const basePath = '/home2/test_envs/<项目名>/frontends/'

    connect()
      .then(() => {
        console.log(chalk.blue('检查备份...'))
        return connExec(`find ${basePath} -maxdepth 1 -name "dist*.tar.gz"`)
      })
      .then(res => {
        if (res) {
          const backFiles = res
            .toString()
            .split('\n')
            .filter(item => item.startsWith(`${basePath}dist`))
            .sort((a, b) => {
              const getTime = file => new Date(file.match(/dist-(.+)\.tar\.gz/)[1].replace(/-/g, '/')).getTime()
              return getTime(b) - getTime(a)
            })
          if (backFiles.length > 2) {
            console.log(chalk.blue('备份超过两个，正在清理多余备份...'))
            const filesToDelete = backFiles.slice(2)
            const deletePromises = filesToDelete.map(file => connExec(`rm -f ${file}`))
            return Promise.all(deletePromises).then(() => {
              console.log(chalk.green('多余备份已清理'))
            })
          }
        } else {
          console.log(chalk.blue('无多余备份'))
          return Promise.resolve()
        }
      })
      .then(() => {
        console.log(chalk.blue('检查dist目录是否存在...'))
        return connExec(`[ -d ${basePath}dist ] && echo 'exists' || echo 'not-exist'`).then(res => {
          if (res && res.toString().includes('exists')) {
            console.log(chalk.blue('开始备份...'))
            const now = new Date()
            const filename = `dist-${now
              .toLocaleString()
              .replace(/\//g, '-')
              .replace(' 上午', 'AM')
              .replace(' 下午', 'PM')
              .replace(' ', '-')}`
            return connExec(`tar -zcPf ${basePath}${filename}.tar.gz ${basePath}dist`)
          } else {
            console.log(chalk.yellow('dist目录不存在，跳过备份'))
            return Promise.resolve()
          }
        })
      })
      .then(() => {
        console.log(chalk.green('备份完成'))
        console.log(chalk.blue('删除远程旧文件...'))
        return connExec(`rm -rf ${basePath}dist`)
      })
      .then(() => {
        console.log(chalk.green('远程删除完成'))
        const spinner = ora('正在发布到服务器...')
        spinner.start()
        scpClient.Client(serverConfig).then(client => {
          client
            .uploadDir('./dist', `${basePath}dist`)
            .then(response => {
              spinner.stop()
              client.close()
              console.log(chalk.green('成功发布到服务器!'))
              console.log(chalk.blue('清理本地dist目录...'))
              fsModule.rmSync('./dist', { recursive: true, force: true })
              console.log(chalk.green('本地dist目录已删除'))
            })
            .catch(error => {
              console.log(chalk.red('发布失败'))
            })
        })
      })
      .catch(err => {
        console.error(chalk.red(err))
      })
      .finally(() => {
        conn.end()
      })
  }
)
```

### 流程说明

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | 检查并清理多余备份 | SSH 连接服务器，保留最近 2 份 dist 备份 |
| 2 | 备份现有 dist | 将远程 dist/ 打包为 dist-时间戳.tar.gz |
| 3 | 删除远程 dist | `rm -rf` 清理远程旧文件 |
| 4 | SCP 上传 | 上传本地 dist/ 到服务器 |
| 5 | 清理本地 dist | `fs.rmSync` 删除本地 dist 目录，释放磁盘空间 |

### 依赖

```json
{
  "devDependencies": {
    "chalk": "^5.x",
    "node-scp": "^1.x",
    "ora": "^8.x",
    "ssh2": "^1.x"
  }
}
```

### serverConfig.mjs（gitignore）

```js
export default {
  host: '192.168.1.2',
  port: 27645,
  username: 'alios',
  password: 'xxx',
}
```

远程路径通常为 `/home2/test_envs/<项目名>/frontends/`，按实际项目替换 `<项目名>`。

## zip.cjs（构建后打包）

`build-prod` 构建完成后自动调用 `node zip.cjs`，将 `dist/` 打包为 `dist.zip`，完成后打开文件管理器定位到该文件。

### Scripts 集成

```json
{
  "scripts": {
    "build-prod": "vue-tsc --noEmit && vite build --mode prod && node zip.cjs"
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
