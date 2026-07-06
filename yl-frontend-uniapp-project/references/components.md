# 自研 my-* 组件体系

引领 uni-app 项目**不使用任何第三方移动端 UI 库**（无 uView、无 uni-ui），全部使用自研的 `my-*` 系列组件。

## 组件一览

| 组件 | 风格 | 用途 |
|------|------|------|
| my-header | Options API | 固定顶部导航栏（返回按钮、slot） |
| my-form | Options API | 配置驱动的动态表单（20+ 字段类型） |
| my-detail | Composition API | 单据详情（按 taskTypeCode 渲染不同类型） |
| my-page | Composition API | 无限滚动分页容器 |
| my-picker | Composition API | 枚举/键值对选择器 |
| my-popup | Composition API | 底部弹出层 |
| my-dialog | Composition API | 居中弹窗对话框 |
| my-alert | Composition API | 成功/提示弹窗 |
| my-search | Composition API | 搜索输入框 |
| my-tab | Composition API | Tab 切换栏 |
| my-timeline | Composition API | 审批时间线 |
| my-upload | Options API | 图片/文件上传 |
| my-datetime | Options API | 自定义日期时间选择器 |
| ly-tree | Options API | 树形选择器（单选/多选、懒加载） |
| uni-transition | Options API | 跨平台动画包装 |

> 新组件使用 Composition API + `<script setup lang="ts">`，已有老组件保持原有风格即可。

---

## my-header — 导航栏

固定定位顶部导航栏，替代 uni-app 原生导航栏。

| Props | 类型 | 说明 |
|-------|------|------|
| white | boolean | 白色背景模式 |
| back | boolean | 是否显示返回按钮（默认 true） |
| home | boolean | 是否显示首页按钮 |
| title | string | 标题文字 |

| Slots | 说明 |
|-------|------|
| default | 标题区内容 |
| left | 左侧区域 |
| right | 右侧区域 |

```vue
<my-header white>用车申请</my-header>

<my-header>
  <template #right>
    <view @click="handleMore">更多</view>
  </template>
</my-header>
```

---

## my-form — 配置驱动表单

通过 `FormConfig[]` 数组驱动渲染，是表单页的核心组件。

### 配置接口

```typescript
interface FormConfig {
  label: string          // 标签文字
  prop: string           // 绑定字段名
  type: string           // 控件类型（见下表）
  required?: boolean     // 是否必填
  show?: boolean | (() => boolean)  // 条件显示
  attrs?: Record<string, any>       // 控件额外属性
  options?: string       // 枚举名或 API 路径（select/picker 类型）
  prop1?: string         // 双绑定字段 1（如 daterange 的 startTime）
  prop2?: string         // 双绑定字段 2（如 daterange 的 endTime）
  rules?: any[]          // 验证规则
  onChange?: (value: any) => void  // 值变化回调
}
```

### 支持的 type

| type | 说明 |
|------|------|
| `input` | 文本输入 |
| `number` | 数字输入 |
| `textarea` | 多行文本 |
| `select` | 下拉选择 |
| `radio` | 单选框 |
| `checkbox` | 多选框 |
| `date` | 日期选择 |
| `datetime` | 日期时间选择 |
| `daterange` | 日期范围（需配 prop1/prop2） |
| `picker` | 自定义选择器 |
| `tree-select` | 树形选择 |
| `upload-img` | 图片上传 |
| `upload-file` | 文件上传 |
| `switch` | 开关 |

### 用法

```vue
<script setup lang="ts">
import type { FormConfig } from '@/types/app'

const formConfig: FormConfig[] = [
  { label: '用车部门', prop: 'deptId', type: 'tree-select', required: true },
  { label: '作业类型', prop: 'workType', type: 'picker',
    options: '/app/workType/list', required: true },
  { label: '用车时间', prop: 'useTime', type: 'daterange',
    prop1: 'startTime', prop2: 'endTime', required: true },
  { label: '备注', prop: 'remark', type: 'textarea' },
]
</script>

<template>
  <my-form :config="formConfig" v-model="formData" />
</template>
```

> 部分项目使用 `formItems` 替代 `config`，参考已有项目用法保持一致。

---

## my-detail — 单据详情

根据 `data.taskTypeCode` 自动切换渲染模板，支持多种单据类型。

| 单据类型 | taskTypeCode | 说明 |
|----------|-------------|------|
| 申请单 | 1001001 | 用车申请详情 |
| 调派单 | 1001002 | 派车单详情 |
| 结算单 | 1001003 | 确认单详情 |

### 功能

- 审批流程图片展示
- 审批时间线（内嵌 my-timeline）
- 附件列表

```vue
<my-detail :data="orderData" />
```

---

## my-page — 分页容器

"点击加载更多" 模式的分页组件。

| Props | 类型 | 说明 |
|-------|------|------|
| status | string | `'more'` / `'loading'` / `'noMore'` |

| Events | 说明 |
|--------|------|
| @loadMore | 触发加载更多 |

```vue
<my-page :status="pageStatus" @loadMore="loadMore" />
```

---

## my-timeline — 审批时间线

展示审批历史，每条记录包含：状态图标、审批人、时间、意见。

```vue
<my-timeline :list="approveHistory" />
```

---

## my-upload — 文件上传

支持图片和文档两种模式：

| Props | 类型 | 说明 |
|-------|------|------|
| count | number | 最大上传数量 |
| del | boolean | 是否允许删除 |

```vue
<my-upload v-model="fileList" :count="9" :del="true" />
```

详见 `references/upload.md`。

---

## my-picker — 选择器

枚举或键值对选择器的包装组件。

```vue
<my-picker
  title="作业类型"
  :options="enumOptions"
  v-model="selectedValue"
/>
```

---

## my-dialog — 居中弹窗

```vue
<my-dialog
  :visible.sync="dialogVisible"
  title="确认操作"
  @confirm="handleConfirm"
>
  <view>弹窗内容</view>
</my-dialog>
```

---

## my-popup — 底部弹出层

常用于选择器、操作菜单：

```vue
<my-popup :visible.sync="popupVisible" title="选择类型">
  <view class="popup-content">
    <!-- 内容 -->
  </view>
</my-popup>
```

---

## my-search — 搜索栏

```vue
<my-search
  v-model="keyword"
  placeholder="请输入搜索关键词"
  @search="handleSearch"
/>
```

---

## my-alert — 提示弹窗

操作成功/失败提示，带图标和文字。

```vue
<my-alert :visible.sync="alertVisible" type="success" message="操作成功" />
```

---

## my-datetime — 日期时间选择器

自定义 5 列选择器（年/月/日/时/分），不依赖系统原生选择器。

```vue
<my-datetime v-model="datetime" />
```

---

## my-tab — Tab 切换

```vue
<my-tab :tabs="tabList" v-model="activeTab" />
```

---

## ly-tree — 树形选择器

完整的树形组件：

| 功能 | 说明 |
|------|------|
| 单选/多选 | `check: false` = radio 模式，`true` = checkbox 模式 |
| 展开/收起 | 支持全部展开或按需展开 |
| 懒加载 | 点击节点动态加载子节点 |
| 搜索过滤 | 内置搜索框过滤节点 |

```vue
<ly-tree
  :data="treeData"
  :check="false"
  @onSelected="onNodeSelected"
/>
```

---

## uni-transition — 动画包装

为任意内容添加跨平台过渡动画（fade、slide、zoom）。

```vue
<uni-transition :show="visible" mode-class="fade">
  <view>动画内容</view>
</uni-transition>
```

---

## 图表组件（部分项目）

| 组件 | 用途 | 依赖 |
|------|------|------|
| l-echart | ECharts 渲染容器 | echarts + zrender |
| bar-chart | 柱状图 | l-echart |
| pie-chart | 饼图 | l-echart |

详见 `references/charts.md`。
