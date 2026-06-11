---
name: yl-frontend-pc-project-vue2
description: 引领 PC 前端 Vue 2 项目开发规范。当修改或新增 Vue 2 PC 端页面、组件、mixins、启动配置时使用。适用于 Vue 2 + Vue CLI + Element UI + Vuex + Options API 技术栈。
metadata:
  version: "2026.6.10"
---

# 引领 PC 前端项目 — Vue 2

> 本 Skill 是 yl-frontend-pc-project 系列中 **Vue 2 专用**篇。
> 跨版本通用内容（API 格式、权限理念、枚举双轨制理念、工具函数、部署流程）已抽入 `yl-frontend-pc-project-shared` skill，
> 本 Skill 聚焦 Vue 2 特化实现。

## 适用范围

本 Skill 适用于所有**引领 (yinling) Vue 2 PC 前端项目**。判断一个项目是否适用，看是否同时满足：

1. 技术栈：Vue 2.6 + Element UI + Vuex 3 + Vue Router 3 + Vue CLI 3
2. API 风格：Options API（`data`/`methods`/`computed`/`watch`）+ Mixins
3. 组件特征：存在自研 `DataGrid`、`Panel`、`Dialog` 等公共组件

### 已知适用项目

| 相对路径 | 项目 | Vue / 构建 |
|---------|------|------------|
| `dynamiccost-container/dynamiccost/` | 油田气动态成本 | 2.6 / Vue CLI 3 |

## 技术栈

Vue 2.6 + Vue CLI 3 + Element UI 2.15 + Vuex 3 + Vue Router 3 + SCSS + ECharts 5

## 约束

- **不可修改后端代码**。需要改后端时先说明理由征求同意。
- 前端使用 npm（老项目用 npm，非 pnpm）
- **本项目不使用 TypeScript**，纯 JavaScript
- **本项目不使用 TSX/JSX**，所有组件为 `.vue` SFC + `<template>` 语法
- **无 Composition API / hooks**，代码复用依赖 Mixins + Vuex + Filters
- API 请求通过 mixins 方法封装（`this.getData()`、`this.postData()` 等），无独立 `src/api/` 目录

## 项目目录

```
dynamiccost-container/
├── dynamiccost/              # 前端（Vue 2）
├── yinling-dynamiccost/      # 后端
└── DB/                       # 数据库脚本
```

```
src/
├── main.js                   # 入口
├── router.js                 # 路由配置（单文件）
├── App.vue                   # 根组件
├── assets/                   # 静态资源（fonts/ + img/）
├── components/               # 33 个自研公共组件
├── filters/                  # Vue 全局过滤器
├── store/                    # Vuex 状态管理
│   ├── index.js              # store 入口（自动注册 modules）
│   ├── getters.js            # 全局 getters
│   └── modules/              # user / system / finance
├── styles/                   # 全局样式（variable.scss + noty.css）
├── utils/                    # 工具函数（axios.js, math.js, index.js, vue-notice.js）
└── views/                    # 页面视图（19 个模块）
    └── mixins/               # settingMixin / chartMixin / baseMixin
```

## 启动项与打包配置

### 端口约定

| 模式 | 端口 | 说明 |
|------|------|------|
| `local` | 3012 | 连本地后端电脑 |
| `onlineTest` | 9012 | 连测试环境 |

### 命令

| 命令 | 说明 |
|------|------|
| `npm run dev` | 默认开发（连本地后端） |
| `npm run dev:local` | 本地开发 |
| `npm run dev:onlineTest` | 连接测试环境 |
| `npm run build` | 打包到 dist/ |
| `npm run build-prod` | 生产打包 + zip |

### .env 文件（VUE_APP_* 前缀）

| 文件 | 触发命令 |
|------|----------|
| `.env.development` | `dev`（默认） |
| `.env.dev-local` | `dev:local` |
| `.env.dev-onlineTest` | `dev:onlineTest` |
| `.env.production` | `build` |
| `.env.prod` | `build-prod` |

## 参考文档

开发时按需查阅。**跨版本通用内容**（API 格式、权限理念、枚举体系理念、工具函数、部署流程）请查阅 `yl-frontend-pc-project-shared` skill。

| 文档 | 内容 |
|------|------|
| `references/architecture.md` | Vue 2 架构：Options API + Mixins + Filters、自研组件体系、深色主题设计 |
| `references/components.md` | 核心组件：DataGrid（自研表格）、Panel（面板）、Dialog（弹窗）、Pagination 等 |
| `references/mixins.md` | Mixins 体系：settingMixin（CRUD+分页+导入导出）、chartMixin、baseMixin |
| `references/page-template.md` | 页面开发模板：setting 设置页标准写法、列表页、详情页模板 |
| `references/startup-config.md` | 启动项配置：vue.config.js 模板、VUE_APP_* 环境变量、vue-cli-service 命令 |
| `references/auth-detail.md` | Vuex 权限实现：store 模块结构、vuex-persistedstate、token/菜单/用户管理 |
| `references/store-patterns.md` | Vuex 模式：namespaced 模块化、自动注册 require.context、持久化配置 |
| `references/axios-interceptors.md` | API 封装：axios 实例（拦截器 + token 注入 + noty 错误提示） |
| `references/data-grid.md` | DataGrid 详解：固定列、行合并、行编辑、多选、汇总行、虚拟滚动 |
| `references/style-dark-theme.md` | 深色主题规范：SCSS 变量体系（variable.scss）、组件配色约定 |
