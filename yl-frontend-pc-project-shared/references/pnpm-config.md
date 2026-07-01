# pnpm 配置指南

> 每个前端项目必须包含自己的 `.npmrc`，**不要依赖全局 pnpm 配置**，确保团队成员行为一致。

## 项目 `.npmrc` 基础配置

```ini
node-linker=isolated
```

### node-linker 说明

| 值 | 行为 | node_modules 结构 |
|----|------|-------------------|
| `isolated`（默认） | 只有直接依赖放在顶层，其余在 `.pnpm/` 内通过符号链接引用 | 包数量少，层级浅 |
| `hoisted` | 所有依赖扁平提升到顶层 （类 npm） | 包数量多，层级深 |

- **推荐 `isolated`**：依赖隔离更严格，能提前发现幽灵依赖问题
- **只有特殊情况才用 `hoisted`**：如工具链兼容性不足

---

## git 依赖 `#` 路径兼容性

当 `package.json` 中有来自 git 仓库的依赖时：

```json
"vue3-treeselect": "git+http://git.example.com/user/vue3-treeselect.git"
```

### 问题

pnpm 在 `isolated` 模式下，`.pnpm` 目录路径中会包含 `#`（commit hash），浏览器/Vite 解析 URL 时把 `#` 视为锚点截断路径，导致 CSS/JS 加载失败。

### 解决方案

| pnpm 版本 | 处理方式 |
|-----------|----------|
| **>= 10.0.0** | 已内置修复，`#` 会被转义，**直接用 `isolated`** |
| **< 10.0.0** | 降级使用 `node-linker=hoisted` 绕过 `.pnpm` 路径问题 |

**建议全部项目升级到 pnpm >= 10**。

---

## 其他常用配置

```ini
# 镜像源（可选，加速下载）
registry=https://registry.npmmirror.com

# 对特定包做提升（精确控制，不推荐全量 hoist）
public-hoist-pattern[]=eslint-*
public-hoist-pattern[]=*prettier*

# store 路径（CI/共享环境使用）
store-dir=/custom/pnpm-store
```

---

## pnpm 版本建议

- 最低要求：**>= 10.0.0**（内置 `#` 转义修复 + SHA256 hash + 安全提升）
- 升级命令：`npm install -g pnpm@10`（有 corepack 时用 `corepack prepare pnpm@latest --activate`）
