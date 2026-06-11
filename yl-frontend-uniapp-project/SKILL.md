---
name: yl-frontend-uniapp-project
description: 引领 uni-app 移动端项目开发规范。当修改或新增 uni-app 页面、组件、hooks、启动配置时使用。适用于 uni-app + Vue 3 + Pinia + 自研 my-* 组件 + 条件编译技术栈。
metadata:
  version: "2026.6.10"
---

# 引领 uni-app 移动端项目

> 本 Skill 是 yl-frontend-pc-project 系列中 **uni-app 移动端专用**篇。

## 适用范围

本 Skill 适用于所有**引领 (yinling) uni-app 移动端项目**。判断一个项目是否适用，看是否同时满足：

1. 技术栈：uni-app + Vue 3 + Pinia + TypeScript
2. 组件特征：使用自研 `my-*` 系列组件（非 uView / uni-ui）
3. 请求方式：`uni.request` + `uni.addInterceptor` 全局拦截器

### 已知适用项目

| 相对路径 | 项目 | uni-app 版本 |
|---------|------|-------------|
| `budget-admin-container/budget-app/` | 靖边数字化财务管控 | 3.0.0 |
| `budget-bt-admin-container/budget-bt-app/` | 宝塔预算 | 3.0.0-alpha |
| `costfeecontrol-admin-container/costfeecontrol-app/` | 成本费用管控 | 3.0.0-alpha |
| `vehicle-admin-container/vehicle-app-new/` | 车辆调派 | 2.0.2 |

## 技术栈

uni-app + Vue 3 + Pinia + TypeScript + SCSS + 自研 my-* 组件

## 目录结构

uni-app 项目**没有传统的 `src/` 目录**，源码直接在项目根目录：

```
<project>-app/
├── App.vue              # 应用入口（Options API，处理生命周期和登录）
├── main.ts              # createSSRApp + Pinia
├── pages.json           # 页面路由 + tabBar + 全局样式配置
├── manifest.json        # 应用配置（AppID、权限、平台配置）
├── pages/               # 页面视图
├── components/          # 公共组件（my-* 系列）
├── stores/              # Pinia store（单 app 模块）
├── utils/               # http.ts（请求封装）、index.ts（工具函数）
├── enums/               # 枚举文件
├── hooks/               # Composition API hooks
├── types/               # TypeScript 类型定义
└── static/              # 静态资源
```

## 约束

- **不可修改后端代码**。需要改后端时先说明理由征求同意。
- 前端使用 pnpm
- 所有项目基于 **Vue 3 + TypeScript**，App.vue 使用 Options API（处理 uni-app 生命周期）
- **不使用第三方移动端 UI 库**（无 uView、无 uni-ui），全部使用自研 `my-*` 组件
- API 请求不分层，直接在各页面中 `import { http } from '@/utils/http'` 内联调用

## 参考文档

| 文档 | 内容 |
|------|------|
| `references/architecture.md` | uni-app 架构：目录布局、条件编译、页面组织、多端适配 |
| `references/components.md` | 自研 my-* 组件体系：my-form、my-dialog、my-page、my-popup、my-search、my-upload 等 |
| `references/hooks.md` | 核心 Hooks：usePage、useEnum、useDoc、useCurrentFP、useDeptTeamDefault |
| `references/page-template.md` | 页面开发模板：列表页、表单页、详情页标准写法 |
| `references/api-patterns.md` | API 请求模式：uni.request 封装、全局拦截器、请求队列管理 |
| `references/auth.md` | 认证与权限：OAuth2 登录、钉钉集成、Pinia store（persist to storageSync） |
| `references/startup-config.md` | 启动项配置：manifest.json、pages.json（tabBar/subPackages）、条件编译环境变量 |
| `references/enums-detail.md` | 枚举体系：uni-app 中的枚举定义与使用模式 |
| `references/condition-compilation.md` | 条件编译规范：#ifdef APP-PLUS / H5 / MP-WEIXIN / MP-ALIPAY |
| `references/upload.md` | 文件/图片上传：uni.chooseImage + uni.uploadFile 封装 |
| `references/charts.md` | ECharts 集成：l-echart 组件、uni-app 中的图表渲染 |
| `references/multi-platform.md` | 多端差异处理：支付宝小程序/H5/APP-PLUS、版本更新、极光推送 |
| `references/pitfalls-and-tips.md` | 常见陷阱与调试：路由栈溢出、TabBar 陷阱、组件通信、性能优化、样式问题 |
| `references/build-and-subpackage.md` | 构建与分包：分包配置、预加载策略、静态资源优化、打包产物分析 |
| `references/navigation-and-lifecycle.md` | 导航与生命周期：App/Page 生命周期、路由导航、页面栈管理、常见场景 |
| `references/storage-and-cache.md` | 存储与缓存：uni.storage、Pinia persist、缓存策略、离线暂存、敏感数据安全 |
