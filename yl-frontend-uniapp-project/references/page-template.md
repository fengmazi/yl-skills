# uni-app 页面开发模板

## 文件书写顺序

```vue
<template>
  <!-- 模板 -->
</template>

<script setup lang="ts">
// Composition API 逻辑
</script>

<style lang="scss" scoped>
/* 样式 */
</style>
```

---

## 列表页标准模板

```vue
<template>
  <my-page
    ref="pageRef"
    :loading="loading"
    :finished="finished"
    @refresh="onRefresh"
    @loadMore="onLoadMore"
  >
    <view class="list-page">
      <my-search v-model="keyword" @search="onRefresh" />

      <view class="list" v-if="list.length">
        <view
          v-for="item in list"
          :key="item.id"
          class="list-item"
          @click="goDetail(item.id)"
        >
          <view class="item-title">{{ item.name }}</view>
          <view class="item-info">{{ item.status }}</view>
        </view>
      </view>

      <view v-else-if="!loading" class="empty">
        暂无数据
      </view>
    </view>
  </my-page>
</template>

<script setup lang="ts">
import { usePage } from '@/hooks/usePage'

const { list, loading, finished, onRefresh, onLoadMore } = usePage('/api/list')

const goDetail = (id: string) => {
  uni.navigateTo({ url: `/pages/detail/detail?id=${id}` })
}
</script>
```

---

## 表单页标准模板

```vue
<template>
  <view class="form-page">
    <my-header title="新增申请" @back="goBack" />

    <my-form
      ref="formRef"
      :formItems="formItems"
      :formData="formData"
      labelWidth="180rpx"
    />

    <view class="form-footer">
      <button type="primary" @click="handleSubmit" :loading="submitting">
        提交
      </button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { http } from '@/utils/http'
import type { FormItem } from '@/types/app'

const formData = reactive({
  name: '',
  type: '',
  date: '',
  remark: '',
})

const formItems: FormItem[] = [
  { label: '名称', prop: 'name', type: 'input', required: true },
  { label: '类型', prop: 'type', type: 'picker', options: [] },
  { label: '日期', prop: 'date', type: 'date' },
  { label: '备注', prop: 'remark', type: 'textarea' },
]

const submitting = ref(false)

const handleSubmit = async () => {
  submitting.value = true
  try {
    await http({ url: '/api/add', method: 'POST', data: formData })
    uni.showToast({ title: '提交成功', icon: 'success' })
    uni.navigateBack()
  } catch {
    uni.showToast({ title: '提交失败', icon: 'error' })
  } finally {
    submitting.value = false
  }
}
</script>
```

---

## 详情页标准模板

```vue
<template>
  <my-page :loading="loading">
    <my-header title="详情" @back="goBack" />

    <view class="detail-page" v-if="detail">
      <!-- 基本信息 -->
      <view class="section">
        <view class="section-title">基本信息</view>
        <view class="info-row">
          <text class="label">名称</text>
          <text class="value">{{ detail.name }}</text>
        </view>
        <view class="info-row">
          <text class="label">状态</text>
          <text class="value">{{ enums.statusMap[detail.status] }}</text>
        </view>
      </view>

      <!-- 审批流程 -->
      <view class="section" v-if="flowSteps.length">
        <view class="section-title">审批流程</view>
        <approve :steps="flowSteps" />
      </view>
    </view>
  </my-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import { http } from '@/utils/http'

const id = ref('')
const detail = ref<any>(null)
const loading = ref(false)

onLoad((options) => {
  id.value = options?.id || ''
  loadDetail()
})

const loadDetail = async () => {
  loading.value = true
  try {
    const res = await http({ url: '/api/detail', data: { id: id.value } })
    detail.value = res.data
  } finally {
    loading.value = false
  }
}
</script>
```

---

## 文件命名约定

| 类型 | 格式 | 示例 |
|------|------|------|
| 页面文件 | 小驼峰 .vue | `apply.vue`, `detail.vue` |
| 页面目录 | 小驼峰 | `task/`, `vehicle/` |
| 组件 | my- 前缀 + 小写连字符 | `my-form`, `my-dialog` |
| Hooks | use 前缀 + 大驼峰 | `usePage`, `useEnum` |
| Store | 小写 | `app.ts` |
| 工具 | 小写 | `http.ts`, `index.ts` |

---

## 路由跳转

```ts
// 页面跳转
uni.navigateTo({ url: '/pages/detail/detail?id=123' })

// 重定向
uni.redirectTo({ url: '/pages/login/login' })

// 返回
uni.navigateBack({ delta: 1 })

// Tab 切换
uni.switchTab({ url: '/pages/index/index' })
```
