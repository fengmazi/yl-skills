# 编码规范 & 编码约定

## 文件命名

| 类型 | 命名风格 | 示例 |
|------|---------|------|
| 页面文件 | camelCase | `applyList.vue` |
| 页面目录 | camelCase | `task/`, `common/` |
| 组件文件 | kebab-case | `my-header`, `my-form` |
| 工具函数 | camelCase | `http.ts` |
| 枚举常量 | camelCase | `approveStatus.ts` |
| 类型定义 | camelCase | `app.d.ts` |
| Hooks | use 前缀 + camelCase | `usePage.ts` |
| Store | 小写 | `app.ts` |

---

## Vue 组件编写规范

### 新增组件：使用 Composition API

```vue
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import { http } from '@/utils/http'
import { useAppStore } from '@/stores'

interface Props {
  id?: string
  mode?: 'create' | 'edit'
}

const props = withDefaults(defineProps<Props>(), {
  mode: 'create',
})

const emit = defineEmits<{
  submit: [data: Record<string, any>]
}>()

const formData = ref<Record<string, any>>({})
const loading = ref(false)

const handleSubmit = async () => {
  loading.value = true
  try {
    await http({ url: '/app/xxx/save', method: 'POST', data: formData.value })
    emit('submit', formData.value)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <view class="page">
    <my-header>标题</my-header>
    <!-- page content -->
  </view>
</template>

<style lang="scss" scoped>
.page {
  // ...
}
</style>
```

### 已有组件：保持原有风格

部分早期组件使用 Options API，修改时保持一致即可，不必重写。

---

## uni-app 标签规范

必须使用 uni-app 标签，**禁止使用原生 HTML 标签**：

| 错误 | 正确 |
|------|------|
| `<div>` | `<view>` |
| `<span>` / `<p>` | `<text>` |
| `<img>` | `<image>` |
| `<a href="">` | `<navigator>` 或 `goPage()` |
| `<input>` | `<input>`（uni-app 内置） |
| `<button>` | `<button>`（uni-app 内置） |

---

## 样式规范

### 单位：使用 rpx

`750rpx = 屏幕宽度`，自动适配不同设备：

```scss
.container {
  width: 690rpx;    // 左右各 30rpx 边距
  padding: 24rpx;
  font-size: 28rpx;
}
```

### 全局样式变量（uni.scss）

```scss
$bg-color: #3b63fe;       // 主题蓝
$danger-color: #ff3d3d;   // 危险红
$font-color: #141e2a;     // 主文字色
```

### 全局工具类（App.vue 中定义）

| 类名 | 用途 |
|------|------|
| `.page` | 全屏弹性布局 |
| `.form` | 表单容器 |
| `.list` | 列表容器 |
| `.filter` | 筛选区 |
| `.btns` | 底部固定按钮区 |
| `.top-bg` | 顶部渐变背景 |
| `.border-b` / `.border-t` | 上下边框 |

---

## 导航规范

使用项目封装的导航辅助函数（`utils/index.ts`）：

```typescript
import { goPage, goBack } from '@/utils'

// 跳转页面
goPage(`/pages/applyDetail?id=${id}`)
goPage('/pages/common/dept?value=deptId&type=radio')

// 返回上一页
goBack()

// 确认操作（弹窗 → API 调用 → 返回 → 发事件刷新）
import { handleOpt } from '@/utils'
handleOpt('删除', `/app/apply/delete/${id}`, {}, 'DELETE', 'refresh_list')
```

### 页面通信

使用 uni-app 事件总线，避免页面间直接耦合：

```typescript
// 发送方 — 操作完成后通知列表刷新
uni.$emit('refresh_apply_list')

// 接收方 — 页面 onShow 中监听
onShow(() => {
  uni.$on('refresh_apply_list', loadData)
})

// 页面卸载时取消监听
onUnload(() => {
  uni.$off('refresh_apply_list')
})
```

---

## API 调用规范

```typescript
import { http } from '@/utils/http'

// 列表查询（POST + querys/sorts 格式）
const res = await http<any>({
  url: '/app/apply/list',
  method: 'POST',
  data: {
    current: 1,
    pageSize: 20,
    querys: [{ property: 'status', value: 'approved', operator: 'EQUAL' }],
    sorts: [{ property: 'createDate', value: 'DESC' }],
  },
})
const list = res.data.data
const total = res.data.total

// 详情
const res = await http<any>({ url: `/app/apply/get/${id}`, method: 'GET' })
const detail = res.data

// 操作
await http({ url: '/app/apply/submit/123', method: 'PUT' })
```

---

## 通用查询参数格式

后端列表接口统一使用 POST：

```json
{
  "current": 1,
  "pageSize": 20,
  "querys": [
    { "property": "fieldName", "value": "value", "operator": "EQUAL" }
  ],
  "sorts": [
    { "property": "createDate", "value": "DESC" }
  ]
}
```

| 操作符 | 说明 |
|--------|------|
| EQUAL | 等于 |
| LIKE | 模糊匹配 |
| GREATER_EQUAL | 大于等于 |
| LESS_EQUAL | 小于等于 |
| IN | 批量匹配 |

---

## 权限控制

```typescript
import { checkAuth } from '@/utils'

const canApply = checkAuth('app-apply')
const canDispatch = checkAuth('app-dispatch')
const canConfirm = checkAuth('app-confirm')
```

在模板中条件渲染：

```vue
<view v-if="canApply" @click="goToApply">用车申请</view>
```

---

## Prettier 配置

```json
{
  "singleQuote": true,
  "semi": false,
  "tabWidth": 2,
  "printWidth": 120,
  "trailingComma": "es5",
  "arrowParens": "avoid"
}
```

---

## 禁止事项

| 禁止 | 正确替代 |
|------|---------|
| 使用 HTML 标签（div/span/img） | uni-app 标签（view/text/image） |
| 使用 `vue-router` | uni-app API（`uni.navigateTo` / `goPage`） |
| 使用 `localStorage` | `uni.getStorageSync/setStorageSync` |
| 使用 `axios` | `utils/http.ts` 的 `http` 函数 |
| 硬编码 baseURL | `utils/http.ts` 中的统一配置 |
| 在 App.vue 中添加业务逻辑 | 放在 Pinia store 中 |
| 直接修改后端代码 | 通过 API 文档/沟通确定接口 |
