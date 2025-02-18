<template>
  <e-charts
    v-if="dailyStats.data"
    ref="dailyChart"
    :option="dailyChartOptions"
    :theme="settingsStore.chartTheme"
    :style="{ height: width * 0.6 + 'px' }"
    autoresize
  />
</template>

<script setup lang="ts">
import { computed, ComputedRef, ref } from 'vue';
import ECharts from 'vue-echarts';
// import { EChartsOption } from 'echarts';

import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { LineChart, BarChart } from 'echarts/charts';
import {
  DatasetComponent,
  GridComponent,
  LegendComponent,
  TitleComponent,
  TooltipComponent,
  VisualMapComponent,
} from 'echarts/components';

import { TimeSummaryReturnValue } from '@/types';
import { useSettingsStore } from '@/stores/settings';
import { EChartsOption } from 'echarts';
import { useElementSize } from '@vueuse/core';

use([
  BarChart,
  LineChart,
  CanvasRenderer,
  GridComponent,
  DatasetComponent,
  LegendComponent,
  TitleComponent,
  TooltipComponent,
  VisualMapComponent,
]);

// Define Column labels here to avoid typos
const CHART_ABS_PROFIT = 'Absolute profit';
const CHART_TRADE_COUNT = 'Trade Count';

const props = defineProps({
  dailyStats: {
    type: Object as () => TimeSummaryReturnValue,
    required: true,
  },
  showTitle: {
    type: Boolean,
    default: true,
  },
});

const settingsStore = useSettingsStore();

const dailyChart = ref(null);
const { width } = useElementSize(dailyChart);

const absoluteMin = computed(() =>
  props.dailyStats.data.reduce(
    (min, p) => (p.abs_profit < min ? p.abs_profit : min),
    props.dailyStats.data[0]?.abs_profit,
  ),
);
const absoluteMax = computed(() =>
  props.dailyStats.data.reduce(
    (max, p) => (p.abs_profit > max ? p.abs_profit : max),
    props.dailyStats.data[0]?.abs_profit,
  ),
);
const dailyChartOptions: ComputedRef<EChartsOption> = computed(() => {
  return {
    title: {
      text: 'Daily profit',
      show: props.showTitle,
    },
    backgroundColor: 'rgba(0, 0, 0, 0)',
    dataset: {
      dimensions: ['date', 'abs_profit', 'trade_count'],
      source: props.dailyStats.data,
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'line',
        label: {
          backgroundColor: '#6a7985',
        },
      },
    },
    legend: {
      data: [CHART_ABS_PROFIT, CHART_TRADE_COUNT],
      right: '5%',
    },
    xAxis: [
      {
        type: 'category',
      },
    ],
    visualMap: [
      {
        dimension: 1,
        seriesIndex: 0,
        show: false,
        pieces: [
          {
            max: 0.0,
            min: absoluteMin.value,
            color: 'red',
          },
          {
            min: 0.0,
            max: absoluteMax.value,
            color: 'green',
          },
        ],
      },
    ],
    yAxis: [
      {
        type: 'value',
        name: CHART_ABS_PROFIT,
        splitLine: {
          show: false,
        },
        nameRotate: 90,
        nameLocation: 'middle',
        nameGap: 35,
      },
      {
        type: 'value',
        name: CHART_TRADE_COUNT,
        nameRotate: 90,
        nameLocation: 'middle',
        nameGap: 30,
      },
    ],
    series: [
      {
        type: 'line',
        name: CHART_ABS_PROFIT,
        // Color is induced by visualMap
      },
      {
        type: 'bar',
        name: CHART_TRADE_COUNT,
        itemStyle: {
          color: 'rgba(150,150,150,0.3)',
        },
        yAxisIndex: 1,
      },
    ],
  };
});
</script>

<style lang="scss" scoped>
.echarts {
  min-height: 240px;
}
</style>
