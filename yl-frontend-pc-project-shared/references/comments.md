# 代码注释规范（最高优先级）

> **铁律**：所有新增/修改的代码，必须同步添加符合本规范的注释。注释不是附加项，是代码的一部分。

## 一、核心理念

注释的目标：**让未来的队友（包括你自己）不骂你**。代码会告诉你「做了什么」，但注释要回答：「为什么这么做」和「怎么用它」。

### 注释的三种动机

| 类型 | 回答的问题 | 典型场景 |
|------|-----------|---------|
| **使用注释**（Praxis） | 怎么用这个函数/组件？ | JSDoc/TSDoc 标注参数、返回值、副作用 |
| **维护注释**（Maintenance） | 为什么这样写？未来怎么改？ | TODO、SHAME、特殊处理说明、技术债标记 |
| **解释注释**（Exposition） | 这段代码在做什么？业务含义？ | 业务规则、算法原理、非自明逻辑 |

### 什么不该注释

- **代码本身已经说明的信息**：`// 获取用户信息` 下面跟 `const user = getUserInfo()` — 废话
- **Git 能回答的信息**：谁写的、什么时候写的、哪个分支 — 不需要
- **外部文档链接写死**：用 ticket 号（如 `DVMP-1234`）代替 URL
- **大段被注释掉的代码**：直接删掉，有 Git 历史

---

## 二、JSDoc 规范（使用注释）

本项目采用 **JSDoc** 标准。TypeScript 已提供类型信息，但 **JSDoc 描述仍然必须写** — 类型告诉你形参是什么类型，JSDoc 告诉你这个参数的业务含义。

### 2.1 通用格式

```ts
/**
 * 简述功能（必填，中文）
 *
 * 详细说明（可选，仅复杂逻辑需要）
 *
 * @param paramName - 参数说明（必填，每个参数都写）
 * @param paramName - 参数说明
 * @returns 返回值说明（必填，无返回值写「无」）
 *
 * @example
 * // 简单示例（可选，工具函数推荐写）
 * useXxx({ id: 1, name: 'test' })
 */
```

### 2.2 Composable Hook（最高优先级，当前最缺失）

**规则：每个 `export default function useXxx()` 必须有 JSDoc。**

```ts
/**
 * 审批/弃审/红冲/撤销操作对话框
 *
 * 根据 opt 类型自动切换表单配置：
 * - '审批'：显示审批表单（通过/驳回） + 详情 + 审批历史
 * - '弃审'：显示弃审原因表单
 * - '撤销'：显示撤销原因表单 + 附件列表
 * - '红冲'：显示红冲表单
 *
 * @param refreshTable - 操作成功后刷新表格的回调
 * @returns { dialogVisible, formConfig, formData, handleOpt, handleCancel }
 *   - dialogVisible: 对话框显示状态
 *   - handleOpt: 触发操作对话框 `(opt: OptProps) => void`
 *   - handleCancel: 关闭对话框
 */
export default function useOperate(refreshTable?: Function) { ... }
```

**返回值对象中每个 key 都必须说明用途。**

### 2.3 工具函数

```ts
/**
 * 将树状数据扁平化为下拉选项数组
 *
 * @param treeData - 树状原始数据
 * @param id - 用作 value 的字段名
 * @param label - 用作 label 的字段名
 * @param children - 子节点字段名，默认 'children'
 * @param disabledFunction - 可选，判断选项是否禁用的回调，入参为当前节点
 * @returns 扁平化的 { value, label, disabled }[] 数组
 */
export function treeDataToFlatOption(
  treeData: VO[],
  id: string,
  label: string,
  children = 'children',
  disabledFunction?: Function
): selectOption[] { ... }
```

### 2.4 TypeScript 类型/接口

```ts
/** 审批操作参数 */
type OptProps = {
  /** 操作类型：审批 | 弃审 | 撤销 | 红冲 */
  opt: optType
  /** 当前行数据 ID，批量操作时传逗号分隔的 ID 串 */
  rowId: string
  /** 接口地址 */
  url: string
  /** 详情请求参数 { businessId, taskTypeCode } */
  param: DetailParam
  /** 是否批量操作，默认 false */
  multiple?: boolean
}
```

**每个属性必须有 `/** 说明 */`**，枚举/联合类型要列举所有可能值。

### 2.5 Vue SFC 组件（`.vue` 文件）

```vue
<script setup lang="ts">
/**
 * 招标订单管理页
 *
 * 功能：订单列表查询、批量送审、批量弃审、订单详情查看、受理/驳回操作
 */
// 后续代码...
```

**Handler 函数也需要 JSDoc：**

```ts
/**
 * 批量送审处理
 * @param rows 勾选的行数据数组
 */
const handleBatchSubmit = (rows: VO[]) => { ... }
```

### 2.6 TSX 组件

```tsx
/**
 * 审批对话框
 *
 * 根据 operationType 区分：
 * - '审批'：显示通过/驳回按钮 + 审批意见输入框
 * - '查看'：只读模式，隐藏操作按钮
 */
export default defineComponent({ ... })
```

### 2.7 Pinia Store

```ts
export const useAppStore = defineStore('app', () => {
  /** 当前登录用户信息 */
  const userInfo = ref<UserInfo | null>(null)

  /** 用户权限码集合，用于 checkAuth() 判断 */
  const permissionCodes = ref<string[]>([])

  /**
   * 检查当前用户是否拥有指定权限
   * @param code - 权限码
   * @returns 是否拥有权限
   */
  const checkAuth = (code: string): boolean => { ... }

  return { userInfo, permissionCodes, checkAuth }
})
```

---

## 三、维护注释

### 3.1 TODO 注释

标记需要后续处理的工作，**必须**包含 ticket 号或明确执行条件。

```ts
// @TODO: 此处硬编码了部门 ID，等接口 /dept/current 上线后替换为动态获取 DVMP-5678
const deptId = '1001'
```

```ts
// @TODO: 临时方案，等 Element Plus 2.5 发布后改用官方虚拟滚动组件
import { VirtualList } from '@/components/common'
```

### 3.2 SHAME 注释

标记明知不符合最佳实践但不得已而为之的代码，**必须**解释原因。

```ts
// @SHAME: 此处用 !important 是因为 cdn.chaos/styles.css 中的 `.menu a` 也有 !important，
// 在公司统一样式库迁移之前无法避免，不要继续在此基础之上增加样式复杂度
.menu .link { color: #00ff00 !important; }
```

```ts
// @SHAME: z-index 9999 是为了覆盖第三方地图组件的弹出层，
// 等地图 SDK 升级到 3.0 后有原生层级配置，届时删除此行
popup.style.zIndex = '9999'
```

### 3.3 PROBLEM / SOLUTION 注释

当前技术限制导致的特殊写法，标注未来可以优化的条件。

```ts
/**
 * 异步 forEach 循环
 *
 * @PROBLEM JavaScript 原生没有异步 forEach，只能用 for...of + await 模拟。
 * 如果未来 ECMAScript 提案 `Promise.forEach` 落地，替换此函数实现。
 *
 * @param array - 要遍历的数组
 * @param callback - 异步回调函数
 */
export async function forEachAsync(array: any[], callback: Function) {
  for (let index = 0; index < array.length; index += 1) {
    // eslint-disable-next-line no-await-in-loop
    await callback(array[index], index, array)
  }
}
```

### 3.4 Linter 禁用注释

禁用 lint 规则时**必须**解释原因。

```ts
// 此函数本身就是异步 forEach 实现，await-in-loop 是其核心逻辑
// eslint-disable-next-line no-await-in-loop
await callback(array[index], index, array)
```

```ts
// 动态导入路径由构建时变量拼接，@typescript-eslint 无法静态分析
// eslint-disable-next-line @typescript-eslint/no-unsafe-member-access
const module = await import(`@/views/${path}`)
```

---

## 四、项目特有约定

### 4.1 区域分隔注释（SFC 页面）

长页面（> 200 行）必须用区域分隔注释组织代码：

```ts
// ==================== 表格数据管理 ====================
const tableData = ref([])
const handleTableRefresh = () => { ... }

// ==================== 新增/编辑对话框 ====================
const addDialogVisible = ref(false)
const handleAdd = () => { ... }

// ==================== 审批对话框 ====================
const approveDialogVisible = ref(false)
const handleApprove = () => { ... }

// ==================== 表格列配置 ====================
const columns: Column[] = [ ... ]
```

### 4.2 配置驱动 UI 注释

`FormItem[][]` 和 `Column[]` 配置数组需要在顶部标注对应的业务场景：

```ts
/** 招标订单查询表单配置 */
const searchFormConfig = computed<FormItem[][]>(() => [ ... ])

/** 招标订单新增/编辑表单配置 */
const editFormConfig = computed<FormItem[][]>(() => [ ... ])
```

### 4.3 迁移后缀说明注释

从其他项目迁入的目录/模块，文件顶部注明来源：

```ts
/**
 * 来源：南泥湾成本管控（nnw）迁入
 * 用途：通用 CRUD 对话框逻辑
 * 修改注意：此文件被宝塔、靖边两个项目共用，修改前请评估影响范围
 */
import { ref } from 'vue'
```

### 4.4 业务逻辑注释

配置型代码（枚举映射、模板映射）必须标注业务含义：

```ts
/** 打印模板类型映射
 *
 * | 编码 | 单据类型 |
 * |------|---------|
 * | 1001 | 招标申请表 |
 * | 1002 | 招标结果审批表 |
 * | 1003 | 合同审批表 |
 */
const templateMap: Record<string, TemplateConfig> = { ... }
```

### 4.5 复杂算法注释

涉及精度计算、特殊转换逻辑的，标注计算规则：

```ts
/**
 * 金额加法（精确计算，避免 IEEE 754 浮点精度问题）
 *
 * 计算规则：将两数乘以 100 转为整数相加后再除以 100
 *
 * @param a - 加数
 * @param b - 被加数
 * @returns 精确的和
 */
export function add(a: number, b: number): number {
  const factor = 100
  return (Math.round(a * factor) + Math.round(b * factor)) / factor
}
```

---

## 五、快速检查清单

写完代码后自查以下 6 条：

| # | 检查项 | 适用场景 |
|---|--------|---------|
| 1 | 每个 `export default function useXxx()` 有 JSDoc | hooks/ |
| 2 | 每个 `export function xxx()` 有 `@param` + `@returns` | utils/ |
| 3 | 每个 `type` / `interface` 属性有 `/** */` | types/ |
| 4 | 每个 store 的 state/getter/action 有注释 | stores/ |
| 5 | 复杂/非常规代码有 TODO / SHAME / PROBLEM 标记 | 全局 |
| 6 | 长页面（>200行）有 `// =====` 区域分隔 | views/ |
