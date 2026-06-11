# 自研 my-* 组件体系

引领 uni-app 项目**不使用任何第三方移动端 UI 库**（无 uView、无 uni-ui），全部使用自研的 `my-*` 系列组件。

## 组件列表

| 组件 | 路径 | 用途 |
|------|------|------|
| my-form | `components/my-form/` | 表单容器，支持配置驱动渲染 |
| my-dialog | `components/my-dialog/` | 弹窗对话框 |
| my-page | `components/my-page/` | 页面容器（下拉刷新、滚动、空状态） |
| my-tab | `components/my-tab/` | 标签页切换 |
| my-header | `components/my-header/` | 页面头部（返回按钮 + 标题） |
| my-popup | `components/my-popup/` | 底部弹出层 |
| my-search | `components/my-search/` | 搜索栏 |
| my-upload | `components/my-upload/` | 图片/文件上传 |
| my-input | `components/my-input/` | 输入框封装 |
| my-picker | `components/my-picker/` | 选择器 |

---

## my-page — 页面容器

最基础、使用频率最高的组件。每个页面都应该用 `my-page` 包裹。

```vue
<template>
  <my-page
    ref="pageRef"
    :loading="loading"
    :finished="finished"
    @refresh="onRefresh"
    @loadMore="onLoadMore"
  >
    <view class="content">
      <!-- 页面内容 -->
    </view>
  </my-page>
</template>
```

| Props | 说明 |
|-------|------|
| loading | 是否加载中 |
| finished | 是否全部加载完成（控制"没有更多了"文案） |
| emptyText | 空状态提示文字 |

| Events | 说明 |
|--------|------|
| @refresh | 下拉刷新回调 |
| @loadMore | 触底加载更多回调 |

---

## my-form — 表单

支持配置驱动渲染，传入 `formItems` 数组自动生成表单。

```vue
<template>
  <my-form
    ref="formRef"
    :formItems="formItems"
    :formData="formData"
    labelWidth="180rpx"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import type { FormItem } from '@/types/app'

const formData = ref({ name: '', type: '' })

const formItems: FormItem[] = [
  { label: '名称', prop: 'name', type: 'input', required: true },
  { label: '类型', prop: 'type', type: 'picker', options: typeOptions },
  { label: '日期', prop: 'date', type: 'date' },
  { label: '备注', prop: 'remark', type: 'textarea' },
]
</script>
```

| FormItem 属性 | 说明 |
|--------------|------|
| label | 标签文字 |
| prop | 绑定字段名 |
| type | 控件类型：input / picker / date / textarea / upload / switch |
| required | 是否必填 |
| options | picker 类型的选项数组 `[{ label, value }]` |
| disabled | 是否禁用 |
| placeholder | 占位文字 |

---

## my-dialog — 弹窗

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

## my-upload — 上传

```vue
<my-upload
  v-model="fileList"
  :maxCount="9"
  action="/api/upload"
  @change="handleUploadChange"
/>
```

详见 `references/upload.md`。

---

## 图表组件（部分项目）

| 组件 | 用途 | 依赖 |
|------|------|------|
| l-echart | ECharts 渲染容器 | echarts + zrender |
| bar-chart | 柱状图 | l-echart |
| pie-chart | 饼图 | l-echart |

详见 `references/charts.md`。
