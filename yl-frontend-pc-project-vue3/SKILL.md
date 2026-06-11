---
name: yl-frontend-pc-project-vue3
description: 引领 PC 前端 Vue 3 项目开发规范。当修改或新增 Vue 3 PC 端页面、组件、hooks、启动配置时使用。适用于 Vue 3 + Vite + Element Plus + vxe-table + Pinia + TypeScript 技术栈。
metadata:
  version: "2026.6.10"
---

# 引领 PC 前端项目 — Vue 3

> 本 Skill 是 yl-frontend-pc-project 系列中 **Vue 3 专用**篇。
> 跨版本通用内容（API 格式、权限理念、枚举双轨制理念、工具函数、部署流程）已抽入 `yl-frontend-pc-project-shared` skill，
> 本 Skill 聚焦 Vue 3 特化实现。

## 适用范围

本 Skill 适用于所有**引领 (yinling) Vue 3 PC 前端项目**。判断一个项目是否适用，看是否同时满足：

1. 技术栈：Vue 3 + Element Plus + vxe-table + Pinia + TypeScript + Vite
2. 架构特征：配置驱动 UI（`FormItem[][]` + `Column[]`）、Hook 组合式开发（`useTable → useFormConfig → useCurd → useDetail/useOperate`）
3. 目录特征：存在 `hooks-bt/`、`hooks/` 或 `components/common-bt/`、`components/common/` 等目录

### 已知适用项目

所有路径相对于工作区根目录（AI 工具打开项目时的根路径）：

| 相对路径 | 项目 | Vue / Vite | 迁移后缀 |
|---------|------|------------|---------|
| `budget-admin-container/budget-admin/` | 靖边预结算 | 3.3 / 4 | `-bt`（宝塔迁入） |
| `budget-bt-admin-container/budget-bt-admin/` | 宝塔预算 | 3.5 / 5 | `-nnw`（南泥湾迁入） |
| `costcontrol-admin-container/costcontrol-admin/` | 南泥湾成本管控 | 3.3 / 4 | —（`-nnw` 源项目） |
| `costfeecontrol-admin-container/costfeecontrol-admin/` | 定边成本管控 | 3.5 / 5 | — |
| `costfeecontrol-zb-admin-container/costfeecontrol-zb-admin/` | 装备制造成本管控 | 3.5 / 5 | — |
| `vehicle-admin-container/vehicle-admin/` | 吴起车辆管理 | 3.5 / 5 | — |
| `eam-admin-vue3-container/eam-admin-vue3/` | 南泥湾资产管理 | 3.5 / 2 | 混合架构（hooks + composables） |

> **版本差异提醒**：Vue 3.3 / Vite 4 项目（靖边、南泥湾成本管控）不能用 `useTemplateRef()` 等 3.5+ API。开发前先确认 `package.json` 中的版本。

### 迁移后缀说明

模块在项目间迁移时，因与原项目代码耦合度高，连带公共代码一起迁入，用后缀区分来源。这个约定适用于 **所有目录类型**：`hooks/`、`components/common/`、`enums/`、`utils/` 等。

**通用规则**：
- 无后缀（`hooks/`、`components/common/`、`enums/`、`utils/`）→ 项目**原有**代码
- 带后缀（`hooks-xxx/`、`components/common-xxx/`、`enums-xxx/`、`utils-xxx/`）→ 从其他项目**迁入**的代码，后缀标识来源

**迁移链**：南泥湾 → 宝塔 (`-nnw`) → 靖边 (`-bt`)。

| 后缀 | 来源项目 | 见于项目 | 示例目录 |
|------|---------|---------|---------|
| `-nnw` | 南泥湾成本管控 | 宝塔预算 | `hooks-nnw/`、`components/common-nnw/`、`enums-nnw/`、`utils-nnw/` |
| `-bt` | 宝塔预算 | 靖边预结算 | `hooks-bt/`、`components/common-bt/`、`enums-bt/` |

**注册顺序**：全局注册时先注册迁入版本再注册原始版本，保证覆盖。如靖边项目 `main.ts` 中先注册 `common-bt` 组件，再注册 `common` 组件。

### 扩展性

目前覆盖**引领**体系项目。如果后续接手其他公司的同类项目（不同 Gogs 组织、不同技术体系），应在 Skill 中新增对应 section，或在项目级 `CLAUDE.md` 中补充差异点。

## 技术栈

Vue 3 + Vite 5 / 4 + Element Plus + vxe-table + Pinia + ECharts + TypeScript

## 约束

- **不可修改后端代码**。需要改后端时先说明理由征求同意。
- 前端使用 pnpm，Git 提交用 commitizen (`pnpm commit`)
- 拉取后端代码注意分支，拿不准先问
- **模板引用使用 `useTemplateRef()`**：Vue 3.5+ 项目获取模板 DOM/组件引用时，使用 `useTemplateRef('refName')` 而非 `ref()`。`useTemplateRef()` 编译时与模板 `ref` 属性绑定，类型更安全。**使用时需先确认项目 Vue 版本 ≥ 3.5**，低于此版本继续用 `ref()`。
  ```ts
  // Vue ≥ 3.5
  const dataTableRef = useTemplateRef('dataTableRef')
  // Options API 组件需加泛型暴露 expose 方法，否则 TS 推断不到
  const dataTableRef = useTemplateRef<{ clearCheckbox: () => void }>('dataTableRef')
  // Vue < 3.5
  const dataTableRef = ref()
  ```
- **批量操作后清除表格勾选**：DataTable 组件已暴露 `clearCheckbox()` 方法。页面批量操作（如批量送审）成功后，必须调用 `dataTableRef.value?.clearCheckbox()` 清除勾选，避免 vxe-grid 的 `reserve: true` 导致已变更状态的行仍被保留在勾选集合中，再次操作时报错。
  ```ts
  // 批量送审成功
  http.post('/xxx/submit', ids).then((res) => {
    ElMessage.success(res.message)
    dataTableRef.value?.clearCheckbox()  // 清除勾选
    handleTableRefresh()
  })
  ```

## 项目目录

每个项目通常包含：

```
<project>-container/        # 仓库根目录
├── <project>-admin/       # 前端 PC
├── <project>-app/         # 前端 APP（可选）
├── yinling-<project>/     # 后端（不修改）
└── DB/                    # 数据库脚本
```

> 使用相对路径引用项目，方便不同开发者/AI 工具共享。

## 启动项与打包配置

### 端口约定

| 模式 | 端口范围 | 说明 |
|------|----------|------|
| `local` | 30xx | 连本地后端电脑 |
| `onlineTest` | 90xx | 连测试环境 |

### 命令常用程度

| 等级 | 命令 | 说明 |
|------|------|------|
| **常用** | `dev:local`、`dev:onlineTest` | 日常开发 |
| **常用** | `build-prod` | 打生产包 |
| **常用** | `deploy` | 部署到测试环境 |
| **常用** | `preview` | 预览构建结果 |
| 很少用 | `dev`、`build` | 默认命令，配置不全且老旧 |

### .env 文件对应

| 文件 | 触发命令 | 用途 | 状态 |
|------|----------|------|------|
| `.env.development` | `dev` | 默认开发 | 很少用，配置老旧 |
| `.env.dev-local` | `dev:local` | 本地开发（连本地后端） | **常用** |
| `.env.dev-onlineTest` | `dev:onlineTest` | 连接测试环境 | **常用** |
| `.env.production` | `build` | 默认打包 | 很少用，配置老旧 |
| `.env.prod` | `build-prod`（`--mode prod`） | 生产环境打包 | **常用** |

### 配置文件关键点

- **Vite**：`vite.config.ts` 通过 `loadEnv(mode, ...)` 读取 `VITE_APP_*` 变量，设置 `server.port` 和 `server.proxy`
- **Vue CLI**：`vue.config.js` 通过 `process.env.VUE_APP_*` 读取变量，设置 `devServer.port` 和 `devServer.proxy`
- 代理目标、端口全部通过环境变量注入，不在配置文件中硬编码
- 详细模板见 `references/startup-config.md`

## 参考文档

开发时按需查阅。**跨版本通用内容**（API 格式、权限理念、枚举体系理念、工具函数、部署流程）请查阅 `yl-frontend-pc-project-shared` skill。

| 文档 | 内容 |
|------|------|
| `references/architecture.md` | Vue 3 架构：配置驱动 UI、Hook 组合式开发、TSX 三种渲染模式、枚举双轨制实现 |
| `references/components.md` | 核心组件 API：DataTable（vxe-grid）、Form、DialogForm、CurdDialog、EditTable、SelectDialog、OptDialog 等 |
| `references/hooks.md` | 核心 Hooks 用法：useTable、useCurd、useFormConfig、useEnum、useDoc、useOperate 等 |
| `references/page-template.md` | 页面开发模板：列表页脚手架、审批页附加层、文件命名约定、filterParam 类型 |
| `references/form-items.md` | 表单配置详解：Element Plus FormItem 完整 type 列表、各类型常用 attrs、动态配置 |
| `references/auth-detail.md` | Pinia 权限实现：checkAuth 权限码规则、useAppStore 关键属性/方法、路由守卫模式 |
| `references/enums-detail.md` | 枚举实现：静态枚举自动 glob 加载（Map/Option/tagTypeMap）、动态枚举 useEnum、使用惯式 |
| `references/startup-config.md` | 启动项配置模板：Vite scripts、.env 文件（VITE_APP_*）、vite.config.ts、deploy.js、zip.cjs |
| `references/dual-system.md` | 双系统共存：common 与 common-xxx 的目录结构、对照表、注册顺序 |
| `references/edittable-scroll.md` | EditTable 横向滚动条跳回问题的原因与修复 |
| `references/pagination-checkbox-reserve.md` | SelectDialog/DialogTable 翻页多选勾选丢失：根因、修复方案、影响分析 |
| `references/selectdialog-groupkey.md` | SelectDialog 按字段分组勾选（groupKey）的用法与实现 |
| `references/print-module.md` | 打印模块架构：目录结构、三段式模板、分页流程、新增单据步骤 |
| `references/copy-record.md` | 整单复制/表头复制：DetailDialog 插槽按钮、copyRecord 模式、适配要点 |
