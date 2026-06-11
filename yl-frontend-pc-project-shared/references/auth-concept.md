# 权限体系理念

> 本页描述引领体系权限体系的**通用设计理念**，与前端框架版本无关。
> 具体实现（Vue 3 Pinia store vs Vue 2 Vuex store）请查阅对应 version skill。

## 权限控制

### 菜单权限

菜单权限在登录时从后端获取，存储在前端状态管理 store 中：
- Vue 3：Pinia `useAppStore()` → `appState.menuAuth`
- Vue 2：Vuex `this.$store.state.user.userMenu`

### 按钮权限检查

使用 `checkAuth` 函数检查用户是否有某个按钮的权限，基本用法：

```ts
import { checkAuth } from '@/utils'

// 条件渲染
checkAuth('xxx-add') && <button onClick={handleAdd}>新增</button>
```

### 权限码规则

权限码通常由页面路径 + 操作组成：

- `xxx-add` — 新增
- `xxx-edit` / `xxx-mod` — 编辑/修改
- `xxx-del` — 删除
- `xxx-export` — 导出
- `xxx-submit` — 提交
- `xxx-check` — 审批
- `xxx-secret` — 保密字段可见

### 多场景权限控制

| 场景 | 用法 |
|------|------|
| 工具栏按钮 | `checkAuth('xxx-add') && <button>新增</button>` |
| 行操作按钮 | 下拉菜单中按权限条件渲染 |
| 列显示权限 | `{ field: 'secret', visible: checkAuth('xxx-secret') }` |
| 角色判断 | `userInfo.isAdmin === 'Y'` 判断管理员 |

## 路由守卫模式

```
用户访问页面
  → 有 Token + 无权限列表？ → 请求导航菜单，提取路由权限
  → 无 Token？ → 跳转登录
  → 路由在权限列表中？ → 放行
  → 不在？ → 跳转 401
```

免验证路由一般包括：`/redirect`、`/401`、`/404`、`/login` 等。

## 登录流程

1. 调用登录接口
2. 保存 Token
3. 获取导航菜单
4. 从导航树提取路由权限列表
5. 保存用户信息、部门列表
