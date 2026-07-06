# PC 端 → APP 端功能同步指南

引领项目通常包含 PC 管理端和 APP 移动端两套前端，共享同一后端服务。当 PC 端新增或修改功能时，需要同步到 APP 端。

## 技术栈对比

| 维度 | PC 端 | APP 端 |
|------|-------|--------|
| 框架 | Vue 3 + vue-router | uni-app (Vue 3) |
| UI 库 | Element Plus / VxeTable | 自研 my-* 组件 |
| HTTP | Axios + 拦截器 | uni.request + 拦截器 |
| 路由 | `src/router/module/*.ts` | `pages.json` |
| 状态管理 | Pinia | Pinia（单 store） |
| 存储 | useLocalStorage | uni.getStorageSync |
| 样式单位 | px / rem | rpx |
| 导航 | router.push() | uni.navigateTo() / goPage() |
| 组件风格 | TSX/JSX | SFC + template |
| 后端接口 | **同一后端服务** | **同一后端服务** |

## 关键组件映射

| PC 端组件 | APP 端等效 |
|-----------|-----------|
| `<router-link>` | `goPage(url)` |
| `router.push()` | `uni.navigateTo()` |
| `router.back()` | `goBack()` / `uni.navigateBack()` |
| `ElTable` / `vxe-grid` 表格 | `<view>` 卡片式列表 |
| `ElForm` / `Form.vue` | `<my-form>` |
| `ElDialog` 弹窗 | `<my-dialog>` |
| `ElDrawer` 抽屉 / 底部弹窗 | `<my-popup>` |
| `Detail.vue` (PC) | `<my-detail>` (App) |
| `Timeline.vue` 时间线 | `<my-timeline>` |
| `useTable` hook | `usePage` hook |
| `ElMessage` 消息 | `uni.showToast()` |
| `ElLoading` 加载 | `uni.showLoading()` |
| `useLocalStorage` | `uni.getStorageSync/setStorageSync` |
| `UploadFile.vue` | `<my-upload>` |
| `SelectDialog.vue` 选择弹窗 | `pages/common/` 选择器页面 + `ly-tree` |

## 同步步骤

### 1. 确认 PC 端改动范围

```bash
# 在 PC 端仓库中查看最近提交
git log --oneline -20
```

确认涉及哪些页面/功能。

### 2. 找到 PC 端 API 调用

PC 端 API 调用常见位置：
- 页面文件：`src/views/*/xxx.vue`
- CRUD hook：`src/hooks/useCurd.ts`
- 表格 hook：`src/hooks/useTable.ts`
- 实体文档 hook：`src/hooks/useDoc.ts`

### 3. APP 端实现

#### 新增页面

1. 在 `pages/` 创建 `.vue` 文件
2. 在 `pages.json` 的 `pages` 数组添加路由
3. 参考同类型既有页面（如新增列表页参考 `applyList.vue`）

#### 调用 API

接口路径一般与 PC 端相同，仅前缀可能有 `/app/` 区分：

```typescript
// PC 端 (axios)
http.get('/vehicleSchedule/apply/list', { params })
http.post('/vehicleSchedule/apply/save', data)

// APP 端 (uni.request)
http({ url: '/app/apply/list', method: 'POST', data: params })
http({ url: '/app/apply/save', method: 'POST', data: formData })
```

### 4. 注意平台差异

| 差异点 | 说明 |
|--------|------|
| 列表展示 | APP 端无表格组件，用 `<view>` 卡片式列表 |
| 分页方式 | PC 用分页器，APP 用无限滚动（`my-page`） |
| 筛选方式 | PC 用表格列筛选，APP 用顶部筛选区 + `my-picker` |
| 表单组件 | PC 用 `Form.vue`，APP 用 `my-form` |
| 弹窗差异 | PC 用 `ElDialog`/`ElDrawer`，APP 用 `my-dialog`/`my-popup` |
| 多端适配 | 需添加条件编译 `#ifdef APP-PLUS` / `#ifdef H5` |

## 常见同步场景

### 列表页新增筛选字段

**PC 端**：`useColumn` 或 `DataTable` 的 columns 配置
**APP 端**：在筛选区添加 `my-picker`，请求 `querys` 中添加对应条件

### 表单页新增字段

**PC 端**：`FormItem[][]` 配置
**APP 端**：在 `FormConfig[]` 中添加对应配置项（`my-form` 驱动）

### 新增审批类型

**PC 端**：`Detail.vue` 中新增 `taskTypeCode` 分支
**APP 端**：`my-detail.vue` 中新增对应 code 的渲染逻辑

### 新增基础数据选择

**PC 端**：`SelectDialog.vue` 或 `SelectTreeDialog.vue`
**APP 端**：在 `pages/common/` 新增选择器页面，使用 `ly-tree` 组件

## 查找后端接口

1. 看 PC 端的网络请求（F12 → Network）
2. 查后端 Swagger 文档（`/swagger-ui.html`）
3. 读后端 Controller 代码
4. 注意：后端代码可能不是最新，先 `git pull`
