---
name: yl-frontend-pc-project-shared
description: 引领 PC 前端项目跨版本通用开发规范。包含 API 请求格式、权限体系理念、枚举双轨制理念、工具函数、部署流程等不依赖具体框架版本的通用内容。当需要了解后端接口规范、权限体系设计、工具函数用法时使用。
metadata:
  version: "2026.6.10"
---

# 引领 PC 前端项目 — 跨版本通用

本 Skill 包含**不依赖 Vue 版本或框架的通用规范**，适用于引领体系所有 PC 前端项目（Vue 2 / Vue 3）。

各版本特化实现请查阅对应的 version skill：
- **Vue 3** → `yl-frontend-pc-project-vue3`（Composition API + Vite + Element Plus + Pinia）
- **Vue 2** → `yl-frontend-pc-project-vue2`（Options API + Vue CLI + Element UI + Vuex）

## 适用范围

本 Skill 适用于所有**引领 (yinling) PC 前端项目**，无论 Vue 2 还是 Vue 3。

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

## 约束（所有版本通用）

- **不可修改后端代码**。需要改后端时先说明理由征求同意。
- 前端使用 pnpm，Git 提交用 commitizen (`pnpm commit`)
- 拉取后端代码注意分支，拿不准先问

## 参考文档

| 文档 | 内容 |
|------|------|
| `references/api-patterns.md` | API 请求模式：分页参数格式、操作符表、CRUD/审批/批量/导出惯用写法 |
| `references/auth-concept.md` | 权限体系理念：权限码规则、角色判断、路由守卫模式（概念层面） |
| `references/enums-concept.md` | 枚举双轨制理念：静态/动态枚举分工、Option/Map 转换、安全回退机制 |
| `references/utilities.md` | 工具函数：formatMoney、smalltoBIG、treeDataToKeyValue、calculateFormula 等 |
| `references/deployment.md` | 部署流程：deploy.js 完整模板、zip.cjs 跨平台打包、serverConfig 配置 |
| `references/textarea-display.md` | 多行文本原样显示（white-space: pre-wrap）的使用场景 |
