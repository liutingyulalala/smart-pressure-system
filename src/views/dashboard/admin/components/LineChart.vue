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
      const { pre_alloc, pred_value, unit_price } = this.reasoningResult
      const { cpu, mem } = unit_price
      if (pre_alloc && pred_value && unit_price) {
        return {
          memData: [pre_alloc?.pod_memory, pred_value?.pod_memory],
          cpuData: [pre_alloc?.pod_cores, pred_value?.pod_cores],
          money: [pre_alloc?.pod_memory * mem + pre_alloc?.pod_cores * cpu, pred_value?.pod_memory * mem + pred_value?.pod_cores * cpu]
        }
      } else {
        return {
          memData: [],
          cpuData: [],
          money: []
        }
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
    setOptions({ memData, cpuData, money } = {}) {
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
          top: 50,
          containLabel: true
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          },
          padding: [5, 10]
        },
        yAxis: [{
          type: 'value',
          name: '配置清单',
          axisTick: {
            show: false
          },
          axisLine: {
            lineStyle: {
              color: '#333'
            }
          }
        },
        {
          type: 'value',
          name: '费用',
          axisTick: {
            show: false
          },
          axisLabel: {
            formatter: '{value} 元'
          },
          axisLine: {
            lineStyle: {
              color: '#333'
            }
          }
        }],
        legend: {
          data: ['内存', 'CPU', '费用']
        },
        series: [
          {
            name: '内存',
            smooth: true,
            barWidth: 80, // 柱状体的宽度
            type: 'bar',
            label: {
              show: true,
              position: 'top',
              backgroundColor: '#e6a700',
              borderRadius: 5,
              padding: [5, 4],
              color: '#fff',
              formatter: '{c} G'
            },
            itemStyle: {
              color: ({ seriesIndex, dataIndex, data, value }) => { console.log(seriesIndex, dataIndex, data, value); return 'rgb(145, 204, 117)' }
            },
            data: memData,
            animationDuration: 2800,
            animationEasing: 'quadraticOut'
          },
          {
            name: 'CPU',
            smooth: true,
            barWidth: 80, // 柱状体的宽度
            type: 'bar',
            label: {
              show: true,
              position: 'top',
              backgroundColor: '#e6a700',
              borderRadius: 5,
              padding: [5, 4],
              color: '#fff',
              formatter: '{c} C'
            },
            itemStyle: {
              color: ({ seriesIndex, dataIndex, data, value }) => { return 'rgb(84, 112, 198)' }
            },
            data: cpuData,
            animationDuration: 2800,
            animationEasing: 'quadraticOut'
          },
          {
            name: '费用',
            type: 'line',
            yAxisIndex: 1,
            lineStyle: {
              type: 'dashed'
            },
            markPoint: {
              label: {
                formatter: '{c} 元'
              },
              data: [
                {
                  name: '最大值',
                  type: 'max'
                },
                {
                  name: '最小值',
                  type: 'min'
                }
              ]
            },
            data: money
          }
        ]
      })
    }
  }
}
</script>
