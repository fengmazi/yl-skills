# 双系统共存（组件/hooks/enums/utils 体系说明）

> **为什么会有两套**：`hooks-nnw/`、`components/common-bt/`、`enums-nnw/`、`utils-nnw/` 等带后缀目录是从其他项目**迁移**过来的代码。迁移时因代码与原项目耦合度高，连带公共代码一起迁入，用后缀区分来源。无后缀的（`hooks/`、`components/common/`、`enums/`、`utils/`）是项目**原有**的代码。后缀因迁移来源而异（`-nnw`=南泥湾、`-bt`=宝塔等），不固定。详见 SKILL.md 中"迁移后缀说明"。

项目可能存在两套体系共存的情况：一套基础体系（项目原有），一套带后缀的迁移体系。

## 判断方式

看 `src/components/` 目录下有哪些子目录：
- 只有 `common/` → 单系统
- 有 `common/` + `common-xxx/` → 双系统共存，参考既有页面风格选择

## 对照表（以 `xxx` 代表后缀）

| 类别 | common 基础体系 | xxx 定制体系 |
|------|---------------|-------------|
| DataTable | `@/components/common/DataTable.vue` | `@/components/common-xxx/DataTable.vue` |
| Hooks 目录 | `@/hooks/` | `@/hooks-xxx/` |
| Utils 目录 | `@/utils/` | `@/utils-xxx/` |
| Enums 目录 | `@/enums/index` | `@/enums-xxx/index` |

## 全局注册顺序

`main.ts` 中先注册 xxx 版本，后注册 common 版本，保证 xxx 同名组件覆盖：

```ts
app.use(xxxComponents)
app.use(commonComponents)
```

## 注意事项

DataTable 的事件名和筛选配置可能因版本不同而有差异，开发时参考已有页面写法保持一致。
