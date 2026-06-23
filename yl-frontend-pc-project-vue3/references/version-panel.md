# VersionPanel — 版本信息调试面板

CTRL+SHIFT+V 唤起弹窗，查看前端构建信息（版本号、Git 提交、打包时间等），无需调用接口。

## 文件

| 文件 | 作用 |
|------|------|
| `src/components/common/VersionPanel.vue` | 版本信息弹窗组件 |
| `vite.config.ts` | `define` 注入 `__APP_VERSION__` / `__APP_GIT_HASH__` / `__APP_GIT_BRANCH__` / `__APP_BUILD_TIME__` |
| `src/vite-env.d.ts` | 全局常量类型声明 |
| `src/App.vue` | 引入并挂载 `<VersionPanel />` |

## 实现步骤

### 1. vite.config.ts — 注入构建变量

```ts
// vite.config.ts
import { readFileSync } from 'fs'
import { execSync } from 'child_process'

const pkg = JSON.parse(readFileSync('./package.json', 'utf-8')) as { version: string }
const gitHash = execSync('git rev-parse --short HEAD').toString().trim()
const gitBranch = execSync('git rev-parse --abbrev-ref HEAD').toString().trim()

export default defineConfig(({ mode }) => {
  return {
    define: {
      __APP_VERSION__: JSON.stringify(pkg.version),
      __APP_GIT_HASH__: JSON.stringify(gitHash),
      __APP_GIT_BRANCH__: JSON.stringify(gitBranch),
      __APP_BUILD_TIME__: JSON.stringify(new Date().toISOString()),
    },
    // ...
  }
})
```

### 2. vite-env.d.ts — 类型声明

```ts
declare const __APP_VERSION__: string
declare const __APP_GIT_HASH__: string
declare const __APP_GIT_BRANCH__: string
declare const __APP_BUILD_TIME__: string
```

### 3. VersionPanel.vue — 组件

组件代码模板（Vue 3 + Element Plus + TypeScript）：

```vue
<template>
  <el-dialog
    v-model="visible"
    title="版本信息"
    width="420px"
    :close-on-click-modal="true"
    :close-on-press-escape="true"
    class="version-panel-dialog"
  >
    <div class="vp-body">
      <div class="vp-row" v-for="item in infoList" :key="item.key">
        <span class="vp-label">{{ item.label }}</span>
        <span class="vp-value" :title="item.value">
          <span class="vp-tag" :class="`vp-tag-${item.level}`" v-if="item.level">
            {{ item.display }}
          </span>
          <span v-else>{{ item.display }}</span>
        </span>
      </div>
      <div class="vp-tip">提示: Ctrl + Shift + V 唤起 / Esc 关闭</div>
      <div class="vp-actions">
        <el-button size="small" @click="copyAll">复制全部</el-button>
        <el-button size="small" type="primary" @click="close">关闭</el-button>
      </div>
    </div>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { ElMessage } from 'element-plus'

const visible = ref(false)

interface InfoItem {
  key: string
  label: string
  value: string
  display: string
  level?: string
}

const formatTime = (iso: string | undefined): string => {
  if (!iso) return '未知'
  try {
    const d = new Date(iso)
    const pad = (n: number) => (n < 10 ? `0${n}` : `${n}`)
    return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`
  } catch { return iso }
}

const parseUA = (ua: string): string => {
  if (!ua) return '未知'
  if (/Edg\//.test(ua)) return 'Edge'
  if (/Chrome\//.test(ua)) return 'Chrome'
  if (/Firefox\//.test(ua)) return 'Firefox'
  if (/Safari\//.test(ua) && !/Chrome/.test(ua)) return 'Safari'
  if (/MSIE|Trident/.test(ua)) return 'IE'
  return ua.slice(0, 30)
}

const infoList = computed<InfoItem[]>(() => [
  { key: 'version', label: '版本号', value: __APP_VERSION__, display: __APP_VERSION__, level: 'version' },
  { key: 'gitHash', label: 'Git 提交', value: __APP_GIT_HASH__, display: __APP_GIT_HASH__ || '未知' },
  { key: 'gitBranch', label: 'Git 分支', value: __APP_GIT_BRANCH__, display: __APP_GIT_BRANCH__ || '未知' },
  { key: 'buildTime', label: '打包时间', value: __APP_BUILD_TIME__ || '', display: formatTime(__APP_BUILD_TIME__) },
  { key: 'env', label: '构建环境', value: (import.meta as any).env?.MODE || '', display: (import.meta as any).env?.MODE || '未知', level: 'info' },
  { key: 'host', label: '访问域名', value: location.host, display: location.host },
  { key: 'url', label: '当前 URL', value: location.href, display: location.href },
  { key: 'ua', label: '浏览器', value: navigator.userAgent, display: parseUA(navigator.userAgent) },
])

const handleKeydown = (e: KeyboardEvent) => {
  const tag = (e.target as HTMLElement)?.tagName || ''
  const editable = (e.target as HTMLElement)?.isContentEditable || /INPUT|TEXTAREA|SELECT/.test(tag)
  if (editable) return
  if (e.ctrlKey && e.shiftKey && (e.key === 'V' || e.key === 'v')) {
    e.preventDefault()
    visible.value = !visible.value
  } else if (e.key === 'Escape' && visible.value) {
    visible.value = false
  }
}

const close = () => { visible.value = false }

const copyAll = async () => {
  const text = infoList.value.map((item) => `${item.label}: ${item.value}`).join('\n')
  try { await navigator.clipboard.writeText(text) }
  catch {
    const textarea = document.createElement('textarea')
    textarea.value = text
    textarea.style.position = 'fixed'
    textarea.style.opacity = '0'
    document.body.appendChild(textarea)
    textarea.select()
    document.execCommand('copy')
    document.body.removeChild(textarea)
  }
  ElMessage.success('已复制到剪贴板')
}

onMounted(() => document.addEventListener('keydown', handleKeydown))
onUnmounted(() => document.removeEventListener('keydown', handleKeydown))
</script>
```

### 4. App.vue — 注册

```vue
<script setup lang="ts">
import VersionPanel from '@/components/common/VersionPanel.vue'
</script>

<template>
  <el-config-provider :locale="zhCn">
    <RouterView />
    <!-- 其他弹窗 ... -->
    <VersionPanel />
  </el-config-provider>
</template>
```

## 样式

组件使用 Element Plus CSS 变量（`--el-text-color-primary`、`--el-border-color-light` 等）和项目 `$main-color` SCSS 变量，自动适配明暗主题。

## 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Ctrl + Shift + V` | 唤起/关闭弹窗 |
| `Esc` | 关闭弹窗（弹窗聚焦时） |

弹窗内部点击遮罩层亦可关闭。

## 注意事项

- `import.meta.env.MODE` 在 Vite 中类型为 `string`，直接访问可能报 TS 类型错误，使用 `(import.meta as any).env?.MODE` 绕过
- Git 信息在构建时通过 `execSync` 获取，CI/CD 环境需确保 `.git` 目录存在
- Vue 2 项目（Options API）版本参考 `dynamiccost-container/dynamiccost/src/components/VersionPanel/index.vue`
