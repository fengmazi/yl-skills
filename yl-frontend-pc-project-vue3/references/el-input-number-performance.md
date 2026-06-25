# el-input-number 输入卡顿问题

## 现象

在 EditTable 子表里用 el-input-number 录入数字时，输入框显示的数字比按键慢，肉眼能感到「卡卡的」。三个数字列（数量、单价、金额）都可能出现。

## 根因

Element Plus 的 `el-input-number` 的 **`change` 事件在每次按键时都会触发**，不是在失焦/回车时才触发。

源码路径（Element Plus 2.x `packages/components/input-number/src/input-number.ts`）：

```ts
// 每次 input 事件都会调到这里
const handleInput = (val: string) => {
  const newVal = val === '' ? null : Number(val)
  if (isNumber(newVal) && !Number.isNaN(newVal)) {
    setCurrentValue(newVal)   // 内部会 emit('change', ...)
  }
}
```

`setCurrentValue` 里同时 emit 了 `input` 和 `change`，所以 `onChange` 回调跟着按键频率跑。

而我们常见的联动写法里，`onChange` 会做这些事：
1. `row.x.mul(row.y)`（big.js 精确乘）
2. `.toFixed(4)` 产出字符串
3. 把字符串赋给另一个被 el-input-number `v-model` 绑定的字段（如 `row.money`）
4. Vue 响应式触发**另一个** el-input-number 重新解析字符串、`precision` 圆整、整列 DOM 更新

第 4 步是同步重活，会卡住 JS 主线程，于是**当前正在输入的那个输入框**显示也跟不上了。

## 解决方案

`onChange` 回调加 **50ms 左右的 setTimeout 防抖**：每次按键重置定时器，短暂停顿后即算联动值。

```tsx
// ❌ 原写法：每次按键都同步算 + 触发跨列重渲染
onChange={() => {
  row.money = row.amount.mul(row.planPrice ? row.planPrice : 0).toFixed(4)
}}

// ✅ 推荐写法：闭包内维护 timer，50ms 节流
slots: {
  edit: ({ row }: { row: any }) => {
    // 防抖：el-input-number 的 change 事件在每次按键时都会触发，
    // 立即同步算金额 + 触发金额列 el-input-number 重渲染会阻塞主线程，
    // 导致当前输入框显示比按键慢。用 50ms 节流合并连续输入。
    let moneyTimer: number | null = null
    return [
      <el-input-number
        v-model={row.amount}
        precision={4}
        onChange={() => {
          if (moneyTimer) clearTimeout(moneyTimer)
          moneyTimer = setTimeout(() => {
            row.money = row.amount.mul(row.planPrice ? row.planPrice : 0).toFixed(4)
          }, 50) as unknown as number
        }}
        // ...
      ></el-input-number>,
    ]
  },
},
```

## 几个注意点

- **闭包 timer 而不是模块级 Map**：`slots.edit` 每个单元格只调用一次，闭包里的 `moneyTimer` 天然就只属于这一行；用 Map 反而要处理清理，得不偿失。
- **每个会触发联动的列都要加**：比如数量→金额、单价→金额、金额→单价三处 onChange 都要分别加 timer，否则只在其中一列加没用。
- **50ms 是经验值**：再小（< 30ms）节流效果不明显，再大（> 200ms）用户能感觉到金额「跟不上手」。实际手感以 50~100ms 之间最自然。
- **别用 `@input` 替代**：`@input` 触发频率更高，且 el-input-number 内部还有 `precision` 圆整等处理，单纯换事件名解决不了问题。
- **TS 里的 setTimeout 返回值**：`setTimeout` 在 Node 类型下返回 `Timeout`，在浏览器返回 `number`，统一 `as unknown as number` 就行。

## 适用范围

- 任何在 EditTable / DataTable 编辑列里用 `el-input-number` + `onChange` 联动的场景
- 表单弹窗里的单个 el-input-number 不会出现这个问题（没有跨列联动开销），不用加

## 相关

- `edittable-scroll.md` — EditTable 横向滚动条跳回问题
