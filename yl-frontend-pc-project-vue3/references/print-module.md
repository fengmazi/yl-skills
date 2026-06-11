# 打印模块架构

> 基于靖边项目 `src/hooks/print/` 的拆分经验，适用于所有引领 PC 项目的打印功能重构。

## 目录结构

```
src/hooks/print/
├── index.ts              # usePrint 入口，编排流程
├── types.ts              # PrintSetting, PrintOption 类型
├── config.ts             # detailMap（单据→API 映射）、列定义常量
├── utils.ts              # getSetting、getStyles、getPrintCalcContainer
├── helpers.tsx            # genTableHeader、genTable、genTableSum、tableTemplate
├── paginate.ts           # buildPrintResult（分页计算）
└── templates/
    ├── index.tsx          # buildTemplates 路由分发
    ├── {taskCode}.tsx     # 每种单据一个模板文件
    └── ...
```

## 各文件职责

| 文件 | 内容 | 可复用性 |
|------|------|---------|
| `types.ts` | `PrintSetting`（A4/针式）、`PrintOption`（单据类型+业务ID） | 通用 |
| `config.ts` | `detailMap`：单据编码 → API URL/方法；列定义常量 | 项目特有 |
| `utils.ts` | `getSetting`、`getStyles`、`getPrintCalcContainer`、`setApproveList` | 通用（纯函数） |
| `helpers.tsx` | `genTableHeader`、`genTable`、`genTableSum`、`tableTemplate` | 通用（纯渲染） |
| `paginate.ts` | `buildPrintResult`：渲染→分页计算→页码拼接 | 通用 |
| `templates/index.tsx` | `preprocessDetail` + `buildTemplates` 路由分发 | 通用框架 |
| `templates/{code}.tsx` | 每种单据的 colgroup + preprocess + build | 项目特有 |

## 模板文件规范

每个模板文件导出 3 个函数：

```tsx
// 1. 列宽配置（纯渲染，不依赖共享 colgroup）
export function colgroup(detail?: Ref<any>): VNode { ... }

// 2. 数据预处理（可选，如字段映射、缺行补齐、动态列获取）
export function preprocess?(detail: Ref<any>, res?: any): any[] | void { ... }

// 3. 构建三段模板
export function build(detail: Ref<any>, base64: string, extraColumns?: any[]): {
  top: VNode       // 每页顶部重复（logo + 单据信息 + 列头）
  detail: VNode    // 数据行（会被分页拆分）
  bottom: VNode    // 仅末页尾部（合计行 + 签字栏）
}
```

### 三段式设计要点

- **top**：放 logo、单据标题、表单信息（单号/日期/部门）、列头。这些内容**每页自动重复**。
- **detail**：只放 `genTable(columns, detail)` 的数据行。分页逻辑会逐行计算高度，超出页面时自动换页。
- **bottom**：放合计行 `genTableSum` 和签字栏。这些内容**只在最后一页**出现，与数据行不分割。

### 新增单据步骤

1. 在 `config.ts` 的 `detailMap` 加 API 映射
2. 创建 `templates/{taskCode}.tsx`，实现 `colgroup` + `build`
3. 在 `templates/index.tsx` 的 `modules` 对象注册一行

## 分页流程

```
usePrint(options)
  → http 查询详情 → detail = Ref(data)
  → preprocessDetail(detail, res, options)   // 数据预处理
  → buildTemplates(options, detail, columns) // 构建三段 VNode
  → renderToString(tableTemplate(VNode, colgroup)) × 3  // VNode → HTML
  → 逐 tr 计算高度，超出 pageHeight 时换页
  → top HTML 每页自动拼接，bottom HTML 仅末页拼接
  → VXETable.print({ content: result })
```

## 空值保护

`genTable` 和 `genTableSum` 需做 null 防护：

```ts
export const genTable = (orderColumns: any[], detail: Ref<any>) => {
  const list = detail.value.isRepair ? detail.value.printDetailList : detail.value.detailList
  if (!list) return <></>
  return (<> {list.map(...)} </>)
}
```

各模板的 `preprocess` 中访问 `detailList` 使用可选链 `?.`。

### big.js 运算空值保护

`preprocess` 中使用 `.add()` `.mul()` `.sub()` `.div()` 等 big.js 拓展方法时，**所有参与运算的值都必须做空值兜底**，否则会抛出 `[big.js] Invalid number`：

```ts
// ❌ 错误：amount/adjustMoney 可能为 null/undefined
item.reMoney = item.money.mul(item.amount)
item.sMoney = item.money.mul(item.amount).add(item.adjustMoney)

// ✅ 正确：所有值使用 ?? 0 兜底
item.reMoney = (item.perPrice ?? 0).mul(item.amount ?? 0)
item.sMoney = (item.perPrice ?? 0).mul(item.amount ?? 0).add(item.adjustMoney ?? 0)
```

**注意**：不要依赖 `item.money = item.perPrice` 中间赋值后再运算，因为 `item.perPrice` 本身也可能为 `null`，应直接用 `item.perPrice ?? 0` 作为运算起点。

## 向后兼容

拆分后保留原始文件作为代理：

```ts
// src/hooks/usePrint.tsx
export { buildPrintResult } from './print'
export type { PrintOption } from './print'
export { default } from './print'
```

确保 `useBatchPrint` 等其他文件无须改动。
