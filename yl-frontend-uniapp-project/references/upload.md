# 文件/图片上传

## 基本用法

使用 uni-app 原生 API 选择文件后上传：

```ts
// 选择图片
const chooseImage = () => {
  uni.chooseImage({
    count: 1,              // 最多选择数量
    sizeType: ['compressed'],  // 压缩图
    sourceType: ['album', 'camera'],  // 相册 + 拍照
    success: (res) => {
      const tempFilePath = res.tempFilePaths[0]
      uploadFile(tempFilePath)
    },
  })
}

// 上传文件
const uploadFile = (filePath: string) => {
  uni.showLoading({ title: '上传中...' })

  uni.uploadFile({
    url: BASE_URL + '/api/upload',
    filePath,
    name: 'file',
    header: {
      Authorization: `Bearer ${appStore.token}`,
    },
    success: (res) => {
      const data = JSON.parse(res.data)
      fileList.value.push({ url: data.url, name: data.name })
    },
    fail: () => {
      uni.showToast({ title: '上传失败', icon: 'error' })
    },
    complete: () => {
      uni.hideLoading()
    },
  })
}
```

---

## my-upload 组件封装

项目中使用 `my-upload` 组件封装上传逻辑：

```vue
<template>
  <my-upload
    v-model="fileList"
    :maxCount="9"
    :action="uploadUrl"
    name="file"
    @change="handleChange"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useAppStore } from '@/stores/modules/app'

const appStore = useAppStore()
const fileList = ref<any[]>([])
const uploadUrl = import.meta.env.VITE_APP_BASE_API + '/api/upload'

const handleChange = (files: any[]) => {
  console.log('文件列表变化:', files)
}
</script>
```

| Props | 说明 |
|-------|------|
| v-model | 已上传的文件列表 |
| maxCount | 最大上传数量 |
| action | 上传接口地址 |
| name | 上传文件的 form 字段名（默认 "file"） |
| accept | 接受的文件类型 |
| disabled | 是否禁用 |

---

## 多平台兼容

```ts
// APP-PLUS / H5: 使用 uni.chooseImage
// #ifdef APP-PLUS || H5
const chooseImage = () => {
  uni.chooseImage({
    count: 1,
    success: (res) => uploadFile(res.tempFilePaths[0]),
  })
}
// #endif

// MP: 使用 uni.chooseMessageFile（小程序专用）
// #ifdef MP
const chooseImage = () => {
  uni.chooseMessageFile({
    count: 1,
    type: 'image',
    success: (res) => uploadFile(res.tempFiles[0].path),
  })
}
// #endif
```

---

## 图片预览

```ts
const previewImage = (url: string) => {
  uni.previewImage({
    urls: [url],       // 预览的图片列表
    current: url,      // 当前显示的图片
  })
}
```

---

## OSS 图片地址拼接（车辆项目）

```ts
// utils/index.ts
export function oss(path: string): string {
  if (!path) return ''
  if (path.startsWith('http')) return path
  return import.meta.env.VITE_APP_OSS_PATH + path
}
```
