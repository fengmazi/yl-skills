# uni-app 枚举体系

## 枚举定义

uni-app 项目中枚举定义在 `enums/` 目录下，每个文件导出一个 key-label 映射对象：

```ts
// enums/approveStatus.ts
export default {
  NEW: '新建',
  SUBMIT: '已提交',
  CHECKING: '审批中',
  PASS: '通过',
  REJECT: '驳回',
  REVOCATION: '撤销',
  ABANDON: '弃审',
}
```

### 手动构建 Option / Map

与 Vue 3 PC 项目不同，uni-app 中枚举文件不会被自动 glob 加载生成 Option/Map，需要手动构造：

```ts
// enums/index.ts — 手动聚合
import approveStatus from './approveStatus'
import applicationStatus from './applicationStatus'

// 构建 Option 数组（供 my-picker 使用）
export const approveStatusOption = Object.entries(approveStatus).map(
  ([value, label]) => ({ value, label })
)

// 构建 Map（供文本显示）
export const approveStatusMap: Record<string, string> = approveStatus

export const enums = {
  approveStatusOption,
  approveStatusMap,
  applicationStatusOption: Object.entries(applicationStatus).map(...),
  applicationStatusMap: applicationStatus,
}
```

---

## 使用方式

### my-picker 中使用

```vue
<template>
  <my-form
    :formItems="[
      { label: '审批状态', prop: 'status', type: 'picker', options: approveStatusOption }
    ]"
  />
</template>

<script setup lang="ts">
import { approveStatusOption } from '@/enums'
</script>
```

### 文本显示

```vue
<text>{{ approveStatusMap[detail.status] || detail.status }}</text>
```

---

## 常见枚举

| 枚举名 | 文件 | 用途 |
|--------|------|------|
| approveStatus | `enums/approveStatus.ts` | 审批状态 |
| applicationStatus | `enums/applicationStatus.ts` | 申请状态 |
| feeType | `enums/feeType.ts` | 费用类型 |
| vehicleStatus | `enums/vehicleStatus.ts` | 车辆状态 |
| orderStatus | `enums/orderStatus.ts` | 订单状态 |
| boolean | `enums/boolean.ts` | 是/否 (Y/N) |
| taskStatus | `enums/taskStatus.ts` | 任务状态 |

---

## 动态枚举

与 PC 端一样，从后端获取业务枚举：

```ts
import { useEnum } from '@/hooks/useEnum'

const { getEnum } = useEnum()

const options = await getEnum(2002)  // groupNo
// → [{ label: 'xxx', value: 'xxx' }]
```
