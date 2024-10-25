<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import { mapState } from 'vuex'
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '350px'
    },
    autoResize: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      chart: null
    }
  },
  computed: {
    ...mapState({
      reasoningResult: state => state.app.reasoningResult
    }),
    chartData() {
      const { pre_alloc, pred_value } = this.reasoningResult
      return {
        expectedData: [20, 30, 40, 50], // [pred_value],
        actualData: [pre_alloc, pred_value]
      }
    }
  },
  watch: {
    chartData: {
      deep: true,
      handler(val) {
        this.setOptions(val)
      }
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')
      this.setOptions(this.chartData)
    },
    setOptions({ expectedData, actualData } = {}) {
      this.chart.setOption({
        xAxis: {
          data: ['当前值', '预测值'],
          boundaryGap: true,
          axisTick: {
            show: false
          },
          axisLine: {
            lineStyle: {
              color: '#333'
            }
          }
        },
        grid: {
          left: 10,
          right: 10,
          bottom: 20,
          top: 30,
          containLabel: true
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          },
          padding: [5, 10]
        },
        yAxis: {
          axisTick: {
            show: false
          },
          axisLine: {
            lineStyle: {
              color: '#333'
            }
          }
        },
        legend: {
          data: ['配置参数']
        },
        series: [
          {
            name: '配置参数',
            smooth: true,
            barWidth: 80, // 柱状体的宽度
            type: 'bar',
            label: {
              show: true,
              position: 'top'
            },
            itemStyle: {
              color: ({ seriesIndex, dataIndex, data, value }) => { return dataIndex === 0 ? '#e6a700' : 'rgb(84, 112, 198)' }
              // normal: {
              //   color: '#ddd',
              //   lineStyle: {
              //     color: '#3888fa',
              //     width: 2
              //   },
              //   areaStyle: {
              //     // color: 'rgb(84, 112, 198)'
              //   }
              // }
            },
            data: actualData,
            animationDuration: 2800,
            animationEasing: 'quadraticOut'
          }]
      })
    }
  }
}
</script>
