# 权限与状态管理

## 权限控制

### 菜单权限

菜单权限在登录时从后端获取，存储在 Pinia store 中：

```ts
import { useAppStore } from '@/stores/app'

const appState = useAppStore()
// appState.menuAuth → ['accountApply-add', 'accountApply-edit', ...]
```

### 按钮权限检查

使用 `checkAuth` 函数检查用户是否有某个按钮的权限：

```ts
import { checkAuth } from '@/utils'

// JSX 工具栏按钮
checkAuth('accountApply-add') && (
  <el-button type="primary" onClick={() => handleAdd('/api/add')}>新增</el-button>
)
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

```ts
// 1. 工具栏按钮
const toolbarBtns = () => [
  checkAuth('xxx-add') && <el-button type="primary" onClick={() => handleAdd('/api/add')}>新增</el-button>
]

// 2. 行操作按钮（dropdown 中）
{
  slots: {
    default: ({ row }) => [
      checkAuth('xxx-edit') && <el-dropdown-item onClick={() => handleEdit(row)}>编辑</el-dropdown-item>,
      checkAuth('xxx-del') && <el-dropdown-item onClick={() => handleDelete(row)}>删除</el-dropdown-item>
    ]
  }
}

// 3. 列显示权限
{ field: 'secret', visible: checkAuth('xxx-secret') }
```

### 角色判断

```ts
const appState = useAppStore()
const isAdmin = appState.userInfo.isAdmin === 'Y'
```

---

## useAppStore 关键属性与方法

| 属性 | 类型 | 说明 |
|-----|-----|-----|
| isCollapse | boolean | 侧边栏折叠状态 |
| navTree | any[] | 导航菜单树 |
| auth | string[] | 路由权限列表 |
| menuAuth | string[] | 菜单按钮权限 |
| deptList | any[] | 部门列表 |
| deptInfo | any[] | 当前部门信息 |
| token | string | 登录 Token |
| userInfo | object | 用户信息 |

**关键方法**：`setAuth()`、`setMenuAuth()`、`setUserInfo()`、`setDept()`、`setToken()`、`clearUserInfo()`

**注意**：不同项目（`-bt` 后缀迁入版本）的 store 路径可能是 `@/stores/app-bt` 或 `@/stores/app`，以实际项目为准。

---

## 路由守卫模式

```ts
router.beforeEach(async (to, from, next) => {
  const appState = useAppStore()

  // 1. 已有 Token 但无权限列表 → 重新获取
  if (appState.auth.length === 0 && appState.token) {
    await http.post('/sysMenu/init').then(res => {
      appState.setNavTree(res.data)
      appState.setAuth(getAuthFromNavTree(res.data))
    })
  }

  // 2. 无 Token → 跳转登录
  if (!appState.token && to.path !== '/login') {
    appState.clearUserInfo()
    return next('/login')
  }

  // 3. 检查路由权限
  if (!noAuthList.includes(to.path) && !appState.auth.includes(to.path)) {
    return next('/401')
  }

  next()
})
```

**免验证路由**一般包括：`/redirect`、`/401`、`/404`、`/login` 等。

---

## 登录流程

1. 调用登录接口
2. 保存 Token → `appState.setToken()`
3. 获取导航菜单 → `appState.setNavTree()`
4. 从导航树提取路由权限 → `appState.setAuth(getAuthFromNavTree())`
5. 保存用户信息、部门列表

```ts
const handleLogin = (formData) => {
  http.post('/login', formData).then(res => {
    appState.setToken(res.data.token)
    appState.setNavTree(res.data.navTree)
    appState.setAuth(getAuthFromNavTree(res.data.navTree))
    appState.setUserInfo(res.data.userInfo)
    appState.setDept(res.data.deptList)
  })
}
```
