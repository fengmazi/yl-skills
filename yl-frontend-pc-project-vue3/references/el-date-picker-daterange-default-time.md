# el-date-picker daterange 选同一天 end time 错误问题

## 现象

查询/筛选区域用 `el-date-picker`（`type="daterange"` + `value-format="x"`）选了**同一天**作为起止时间，发出的请求里 `BETWEEN` 的两个时间戳都是当天 00:00:00：

```json
// 用户在 UI 上选了 2026-06-26 ~ 2026-06-26（只选一天）
{"querys":[{"property":"oilChargeDate","operator":"BETWEEN","value":[1782316800000,1782316800000]}]}
```

后端按这个区间只能查到**当天零点那一刻**的加油记录（实际一条都查不到），整张表查不出数据。选不同天的时候则正常（end 是当天 23:59:59.999）。

## 根因

`el-date-picker` 的 `type="daterange"` 选中两天时，会把结束时间默认设为当天的 **00:00:00**；用户只选一天（start = end）时 start 和 end 都是当天 00:00:00。

Element Plus 没有把这个默认值和「同一天」场景做特殊处理，**`value-format="x"` 会把这个 00:00:00 原样序列化成毫秒戳**，所以请求里 end 就停留在当天 00:00:00。

`:default-time` 属性虽然能改 picker 的默认时间显示，**但不会改写 v-model 的实际值**——所以光加 `default-time` 还不够，还得在请求里显式把 end 推到当天末。

## 解决方案

两步配套使用：

### 1. 模板加 `:default-time`

让 picker 在用户选同一天时，UI 上自动展开成「当天 00:00:00 ~ 当天 23:59:59」，避免误以为选的就是「00:00:00 ~ 00:00:00」。

```html
<el-date-picker
  v-model="querys.oilChargeDate"
  type="daterange"
  range-separator="To"
  start-placeholder="加油开始时间"
  end-placeholder="加油结束时间"
  value-format="x"
  size="small"
  :default-time="defaultTime"
/>
```

### 2. 脚本里定义 `defaultTime` + 显式 setHours end

```ts
// el-date-picker（type="daterange"）默认 start/end 时间均为 00:00:00，
// 选同一天时 v-model 是 [00:00:00, 00:00:00]，BETWEEN 区间查不到当天数据。
// :default-time 仅影响 picker 的默认时间显示，不会改写 v-model 实际值；
// 实际请求的 end 时间必须在 queryList 中通过 setHours(23, 59, 59, 999) 显式修正。
const defaultTime: [Date, Date] = [new Date(0, 0, 0, 0, 0, 0), new Date(0, 0, 0, 23, 59, 59)]

const queryList = computed(() => {
  const list: any = []
  if (querys.oilChargeDate) {
    list.push({
      property: 'oilChargeDate',
      operator: 'BETWEEN',
      // 第四个参数 999 必须显式传：setHours(h, m, s) 不传毫秒时为 0，
      // 会得到 23:59:59.000，与后端期望的 23:59:59.999 差 999ms。
      value: [querys.oilChargeDate[0], new Date(querys.oilChargeDate[1]).setHours(23, 59, 59, 999)],
    })
  }
  // ... 其他字段
})
```

## 几个注意点

- **两步必须都做**。光加 `:default-time` 不写 `setHours`，picker UI 看着正常，请求里 end 还是 00:00:00；光写 `setHours` 不加 `:default-time`，picker 在 UI 上仍然显示「00:00:00 ~ 00:00:00」，用户不知道自己其实只查到了当天零点那一刻。
- **`setHours` 必须传第四个参数 `999`**。`setHours(h, m, s)` 不传毫秒时毫秒位为 0，输出 `23:59:59.000`，比后端期望的 `23:59:59.999` 差 999 毫秒。SQL `BETWEEN` 场景下虽然能查到绝大多数数据，但和后端约定的格式不一致。**项目里旧代码（`deptFuelSummary.vue` / `deptVehicleFuelSummary.vue` / `fuelExSummary.vue` / `vehicleFuelSummary.vue`）都用 `setHours(23, 59, 59)`，属于同款 bug**。
- **`defaultTime` 用固定 `Date` 数组就够**，不要用 fiscalPeriod 返回的 `[startMilliTime, endMilliTime]`（数字数组，类型不匹配 `Date[]`，且会把 end 默认时间锁死成 fiscal period 结束时刻）。项目里有个反例（`deptFuelSummary.vue` 等用 `ref<[Date, Date]>()` + 数字赋值）目前能跑只是因为 Element Plus 内部做了宽松转换，不建议模仿。
- **导出场景也必须 setHours**。`fuelExSummary.vue` 里的 `exportExcel` 直接把 `date.value` 拼到请求里而没有 `setHours`，跨天选同一天导出时会少数据——属于已知 bug，遇到时按本方案同款修。
- **`@change` 事件里也建议校正一次**。picker 的 `@change` 拿到的是 raw value，UI 上看就出错了，调用方很容易信以为真。在 `@change` 里把 end 校正后再写入 `querys[key]`，是最稳的写法；只在请求时校正，picker 显示和实际值会不一致。

## 适用范围

- 任何「`el-date-picker` + `type="daterange"` + `value-format="x"`」的查询/筛选区域
- 弹窗表单里选日期范围虽然不受这个 bug 影响（多数场景下选的都是不同天），但加上 `:default-time` 能让 UI 更明确，没坏处
- vxe-table 自带的日期范围筛选**不受这个 bug 影响**（它内部已经处理过 end 默认时间），不用加

## 相关

- `form-items.md` — `date` 类型小节里有交叉引用
- `el-input-number-performance.md` — 同为 Element Plus 控件的常见坑，结构类似
