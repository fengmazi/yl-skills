# ECharts 图表集成

部分项目（budget-bt-app、costfeecontrol-app）集成了 ECharts 用于数据可视化。

## 依赖

```json
{
  "echarts": "^5.4.3 或 ^6.0.0",
  "zrender": "^6.0.0"
}
```

> zrender 是 ECharts 的底层渲染库，需单独安装。

---

## l-echart 组件

项目中使用 `l-echart` 组件作为 ECharts 渲染容器：

```vue
<template>
  <view class="chart-container">
    <l-echart ref="chartRef" @finished="onChartReady" />
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref()
let chartInstance: any = null

const initChart = () => {
  chartInstance = echarts.init(chartRef.value)

  const option = {
    tooltip: { trigger: 'axis' },
    xAxis: { type: 'category', data: ['1月', '2月', '3月'] },
    yAxis: { type: 'value' },
    series: [{ type: 'bar', data: [120, 200, 150] }],
  }

  chartInstance.setOption(option)
}

const onChartReady = () => {
  initChart()
}

// 页面隐藏时销毁图表
onHide(() => {
  chartInstance?.dispose()
})
</script>

<style lang="scss" scoped>
.chart-container {
  width: 100%;
  height: 400rpx;
}
</style>
```

---

## bar-chart / pie-chart 封装

部分项目封装了 bar-chart 和 pie-chart 组件，简化使用：

```vue
<template>
  <bar-chart :data="barData" />
  <pie-chart :data="pieData" />
</template>

<script setup lang="ts">
const barData = {
  categories: ['A', 'B', 'C'],
  series: [{ name: '数值', data: [10, 20, 30] }],
}

const pieData = {
  series: [{ name: '占比', data: [{ name: 'A', value: 10 }, { name: 'B', value: 20 }] }],
}
</script>
```

---

## uni-app 中使用 ECharts 的注意事项

| 注意点 | 说明 |
|--------|------|
| 初始化时机 | 需要等组件 `@finished` 事件触发后再初始化 |
| 销毁 | `onHide` / `beforeDestroy` 中必须 `dispose()` |
| 尺寸 | 使用 `rpx` 单位，ECharts 内 `resize()` 需手动调用 |
| 小程序 | **小程序不支持 ECharts**，需要用条件编译排除 |
| 事件穿透 | ECharts 的 touch 事件可能被 uni-app 拦截，需要配置 `canvasId` |

---

## 小程序兼容

小程序不支持 ECharts，需要用条件编译排除图表页面：

```json
// pages.json
{
  "subPackages": [
    // #ifndef MP
    { "root": "pages/chart", "pages": [{ "path": "index" }] }
    // #endif
  ]
}
```
