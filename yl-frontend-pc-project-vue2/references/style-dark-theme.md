# 深色主题 SCSS 规范

项目采用暗蓝基调的深色主题设计，所有 SCSS 变量定义在 `src/styles/variable.scss`。

## 核心色板

```scss
// 背景色系
$bg-primary: #0a1a2e;       // 主背景（深蓝黑）
$bg-secondary: #0d2137;     // 次级背景
$bg-panel: #112240;         // 面板背景
$bg-card: #152a45;          // 卡片背景

// 边框色系
$border-color: #1e3a5f;     // 主边框
$border-light: #2a4a7f;     // 浅边框（悬停态）

// 文字色系
$text-primary: #e0e6ed;     // 主文字
$text-secondary: #8899aa;   // 次级文字
$text-muted: #5a6a7a;       // 弱化文字

// 强调色系
$accent-blue: #409eff;      // 主强调（Element UI 蓝）
$accent-cyan: #00d4ff;      // 青色强调（装饰）
$accent-green: #67c23a;     // 绿色（成功）
$accent-orange: #e6a23c;    // 橙色（警告）
$accent-red: #f56c6c;       // 红色（危险）

// 表格色系
$table-header-bg: #0d2137;  // 表头背景
$table-row-bg: #112240;     // 行背景
$table-row-hover: #1a3355;  // 行悬停
$table-row-stripe: #132238; // 斑马纹
```

---

## 使用示例

### 页面背景

```scss
.page-container {
  background: $bg-primary;
  min-height: 100vh;
  padding: 16px;
}
```

### Panel 面板

```scss
.panel {
  background: $bg-panel;
  border: 1px solid $border-color;
  border-radius: 4px;
  margin-bottom: 16px;

  &__header {
    color: $accent-cyan;
    border-bottom: 1px solid $border-color;
    padding: 12px 16px;
    font-size: 14px;
  }

  &__body {
    padding: 16px;
    color: $text-primary;
  }
}
```

### DataGrid 表格

```scss
.data-grid {
  // 表头
  &__header {
    background: $table-header-bg;
    color: $accent-cyan;
    font-weight: 500;
  }

  // 行
  &__row {
    background: $table-row-bg;
    border-bottom: 1px solid $border-color;

    &:hover {
      background: $table-row-hover;
    }

    &:nth-child(even) {
      background: $table-row-stripe;
    }
  }
}
```

### 弹窗

```scss
.dialog {
  background: $bg-panel;
  border: 1px solid $border-light;

  &__header {
    background: $bg-secondary;
    color: $text-primary;
    border-bottom: 1px solid $border-color;
  }

  &__body {
    color: $text-primary;
  }
}
```

---

## Element UI 组件覆盖

通过 SCSS 变量或深度选择器覆盖 Element UI 默认样式：

```scss
// 输入框覆盖
::v-deep .el-input__inner {
  background: $bg-secondary;
  border-color: $border-color;
  color: $text-primary;

  &:focus {
    border-color: $accent-blue;
  }
}

// 下拉菜单覆盖
::v-deep .el-select-dropdown {
  background: $bg-panel;
}

::v-deep .el-select-dropdown__item {
  color: $text-primary;

  &:hover {
    background: $table-row-hover;
  }
}

// 按钮覆盖
::v-deep .el-button--primary {
  background: $accent-blue;
  border-color: $accent-blue;
}
```

---

## 注意事项

1. **所有颜色必须通过变量引用**，不允许硬编码色值
2. 新增组件时优先使用已有变量，确需新变量时添加到 `variable.scss`
3. 覆盖 Element UI 样式使用 `::v-deep` 深度选择器
4. 不要修改 `variable.scss` 中的已有变量值（会影响全局）
