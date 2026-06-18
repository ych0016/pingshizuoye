<template>
  <div class="chart-card">
    <h3>业务环节关联度分析</h3>
    <div ref="chartRef" class="chart-container"></div>
  </div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref, watch, nextTick } from "vue";
import * as echarts from "echarts";

const props = defineProps({
  data: {
    type: Array,
    required: true,
    default: () => [
      { name: "海外运营", value: 9 },
      { name: "海外贸易", value: 5 },
      { name: "海外采购", value: 3 },
      { name: "跨境结算", value: 3 },
      { name: "供应链物流", value: 3 },
      { name: "投资设立", value: 3 },
      { name: "数据合规", value: 2 },
      { name: "总部布局", value: 2 },
    ],
  },
});

const chartRef = ref(null);
let chartInstance = null;
let resizeObserver = null;

// 初始化图表
const initChart = () => {
  if (!chartRef.value) return;
  chartInstance = echarts.init(chartRef.value);
  updateChart(props.data);
};

// 更新图表数据
const updateChart = (dataList) => {
  if (!chartInstance || !Array.isArray(dataList)) return;

  // 按 value 升序排序（和图中顺序一致：从上到下从小到大）
  const sortedData = [...dataList].sort((a, b) => a.value - b.value);
  const yAxisData = sortedData.map((item) => item.name);
  const seriesData = sortedData.map((item) => item.value);

  const option = {
    backgroundColor: "transparent",
    tooltip: {
      trigger: "axis",
      axisPointer: {
        type: "shadow",
      },
      backgroundColor: "rgba(20, 20, 20, 0.88)",
      borderColor: "rgba(255, 255, 255, 0.1)",
      borderWidth: 1,
      textStyle: {
        color: "#ffffff",
        fontSize: 12,
        fontFamily: "inherit",
      },
    },
    grid: {
      left: "18%",
      right: "8%",
      bottom: "3%",
      top: "5%",
      containLabel: true,
    },
    xAxis: {
      type: "value",
      max: 10, // 和图中0-10的刻度完全匹配
      axisLine: { show: false },
      axisTick: { show: false },
      splitLine: {
        lineStyle: {
          color: "rgba(255, 255, 255, 0.05)",
        },
      },
      axisLabel: {
        color: "rgba(255, 255, 255, 0.6)",
        fontSize: 10,
      },
    },
    yAxis: {
      type: "category",
      data: yAxisData,
      axisLine: {
        lineStyle: {
          color: "rgba(255, 255, 255, 0.08)",
        },
      },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(255, 255, 255, 0.72)",
        fontSize: 11,
      },
    },
    series: [
      {
        name: "文献关联数",
        type: "bar",
        barWidth: "42%",
        data: seriesData,
        itemStyle: {
          borderRadius: [0, 4, 4, 0],
          // 金色渐变，和图中条形颜色完全一致
          color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
            { offset: 0, color: "rgba(200, 168, 102, 0.08)" },
            { offset: 1, color: "#c8a866" },
          ]),
        },
      },
    ],
  };

  chartInstance.setOption(option);
};

// 监听数据变化
watch(
  () => props.data,
  (newData) => {
    updateChart(newData);
  },
  { deep: true }
);

onMounted(() => {
  initChart();
  // 确保容器渲染完成后再resize，解决空白问题
  nextTick(() => {
    chartInstance?.resize();
  });

  // 响应式容器大小变化
  if (window.ResizeObserver && chartRef.value) {
    resizeObserver = new ResizeObserver(() => {
      chartInstance?.resize();
    });
    const parent = chartRef.value.parentElement;
    if (parent) resizeObserver.observe(parent);
  }
});

onBeforeUnmount(() => {
  resizeObserver?.disconnect();
  chartInstance?.dispose();
});
</script>

<style scoped>
.chart-card {
  padding: clamp(20px, 3vw, 28px);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.07);
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.025), rgba(255, 255, 255, 0.005)),
    rgba(16, 16, 16, 0.48);
  backdrop-filter: blur(14px);
  box-shadow: var(--shadow-deep);
  display: flex;
  flex-direction: column;
  height: 380px;
}

h3 {
  font-size: 15px;
  font-weight: 900;
  margin: 0 0 16px;
  color: rgba(255, 255, 255, 0.9);
  letter-spacing: 0.05em;
}

.chart-container {
  flex-grow: 1;
  width: 100%;
  height: 100%;
  min-height: 200px; /* 兜底高度，防止容器高度为0导致空白 */
}
</style>