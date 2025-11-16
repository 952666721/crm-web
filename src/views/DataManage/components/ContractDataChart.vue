<template>
  <div id="tradePie" style="height: 400px; width: 100%"></div>
</template>

<script lang="ts" name="ContractPieChart" setup>
import * as echarts from 'echarts'
import { onMounted, ref, onUnmounted, watch } from 'vue'
import { TradeArray } from '@/api/interface/index'
import { PropType } from 'vue'

const props = defineProps({
  data: {
    type: Object as PropType<TradeArray>,
    required: true,
    default: () => ({ timeList: [], countList: [] })
  }
})

const myChart = ref<echarts.ECharts | null>(null)
let resizeHandler: () => void

const initChart = () => {
  // 1. 数据校验（严格处理空数据）
  const timeList = Array.isArray(props.data.timeList) ? props.data.timeList : []
  const countList = Array.isArray(props.data.countList) ? props.data.countList : []

  // 2. 生成饼图数据（确保 value 为有效数字）
  const pieData = timeList
    .map((time, index) => ({
      name: time || `未知时间${index}`, // 避免 name 为 undefined
      value: Number(countList[index]) || 0 // 强制转为数字，避免非数值类型
    }))
    .filter((item) => item.value > 0) // 过滤无效数据

  // 3. 处理无数据场景（显示空图表提示）
  if (pieData.length === 0) {
    if (myChart.value) {
      myChart.value.clear() // 清空图表
      myChart.value.setOption({
        title: { text: '暂无数据', left: 'center', top: 'middle' },
        series: [] // 空 series 避免解析错误
      })
    }
    return
  }

  // 4. 初始化图表实例（确保 DOM 存在）
  const chartDom = document.getElementById('tradePie')
  if (!chartDom) return
  if (myChart.value) myChart.value.dispose()
  myChart.value = echarts.init(chartDom)

  // 新增：定义渐变颜色数组（环形图专用）
  const colorList = [
    {
      type: 'linear',
      x: 0,
      y: 0,
      x2: 1,
      y2: 0,
      colorStops: [
        { offset: 0, color: '#4facfe' },
        { offset: 1, color: '#00f2fe' }
      ]
    },
    {
      type: 'linear',
      x: 0,
      y: 0,
      x2: 1,
      y2: 0,
      colorStops: [
        { offset: 0, color: '#fa709a' },
        { offset: 1, color: '#fee140' }
      ]
    },
    {
      type: 'linear',
      x: 0,
      y: 0,
      x2: 1,
      y2: 0,
      colorStops: [
        { offset: 0, color: '#30cfd0' },
        { offset: 1, color: '#330867' }
      ]
    },
    {
      type: 'linear',
      x: 0,
      y: 0,
      x2: 1,
      y2: 0,
      colorStops: [
        { offset: 0, color: '#84fab0' },
        { offset: 1, color: '#8fd3f4' }
      ]
    },
    {
      type: 'linear',
      x: 0,
      y: 0,
      x2: 1,
      y2: 0,
      colorStops: [
        { offset: 0, color: '#a1c4fd' },
        { offset: 1, color: '#c2e9fb' }
      ]
    }
  ]

  // 5. 修改后的图表配置（环形图样式）
  const option = {
    title: {
      text: '合同数量分布',
      left: 'center',
      textStyle: {
        fontSize: 18,
        fontWeight: 'normal',
        color: '#333'
      },
      subtextStyle: {
        color: '#666'
      }
    },
    tooltip: {
      trigger: 'item',
      formatter: '{a} <br/>{b}: {c} 单 ({d}%)',
      backgroundColor: 'rgba(255, 255, 255, 0.9)',
      borderColor: '#eee',
      borderWidth: 1,
      textStyle: { color: '#333' },
      padding: 10,
      boxShadow: '0 2px 10px rgba(0, 0, 0, 0.1)'
    },
    legend: {
      orient: 'horizontal', // 改为水平布局
      bottom: 10, // 放在底部
      left: 'center',
      itemWidth: 12,
      itemHeight: 12,
      textStyle: { fontSize: 12, color: '#666' },
      data: pieData.map((item) => item.name)
    },
    series: [
      {
        name: '合同数量',
        type: 'pie',
        radius: ['40%', '70%'], // 环形图（内半径40%，外半径70%）
        center: ['50%', '45%'], // 图表居中
        color: colorList, // 使用渐变颜色
        avoidLabelOverlap: false,
        itemStyle: {
          borderColor: '#fff', // 扇区边框白色
          borderWidth: 2,
          borderRadius: 4 // 扇区边角圆角
        },
        label: {
          show: false, // 隐藏扇区内部标签
          position: 'center'
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 16,
            fontWeight: 'bold',
            color: '#333'
          }
        },
        labelLine: {
          show: false // 隐藏标签连接线
        },
        data: pieData
      }
    ]
  }

  myChart.value.setOption(option)

  // 6. 窗口 resize 事件（保持不变）
  resizeHandler = () => myChart.value?.resize()
  window.removeEventListener('resize', resizeHandler)
  window.addEventListener('resize', resizeHandler)
}

// 监听数据变化（保持不变）
watch(
  () => [props.data.timeList, props.data.countList],
  () => initChart(),
  { immediate: true, deep: true }
)

// 组件生命周期管理（保持不变）
onMounted(initChart)
onUnmounted(() => {
  if (myChart.value) {
    myChart.value.dispose()
    myChart.value = null
  }
  if (resizeHandler) {
    window.removeEventListener('resize', resizeHandler)
  }
})
</script>
