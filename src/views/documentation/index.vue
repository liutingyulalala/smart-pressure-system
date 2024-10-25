<template>
  <div class="doc-wrapper">
    <!-- <el-alert
      title="提示"
      type="info"
      description="推理完成2秒后将跳转到价值成果页面"
      show-icon
    /> -->
    <div class="form-container">
      <el-form
        ref="form"
        :model="form"
        label-width="150px"
      >
        <el-form-item label="模型类型">
          <el-radio-group v-model="form.model_cate">
            <el-radio label="cpu">cpu核数</el-radio>
            <el-radio label="throughput">服务吞吐量</el-radio>
            <el-radio label="mem">内存大小</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item v-if="form.model_cate !== 'throughput'" label="服务吞吐量">
          <el-input v-model="form.throughput_req">
            <template slot="append"><span class="unit" style="font-size: 12px;">TPS</span></template>
          </el-input>
        </el-form-item>
        <el-form-item v-if="form.model_cate !== 'throughput'" label="支撑人数">
          <el-input v-model="form.people">
            <template slot="append"><span class="unit" style="font-size: 12px;">人</span></template>
          </el-input>
        </el-form-item>
        <el-form-item v-show="false" label="模型压力">
          <el-input v-model="form.threads_data_Threads">
            <template slot="append"><span class="unit">个</span></template>
          </el-input>
        </el-form-item>
        <el-form-item v-if="form.model_cate !== 'mem'" label="内存大小">
          <el-input v-model="form.pod_memory">
            <template slot="append"><span class="unit">G</span></template>
          </el-input>
        </el-form-item>
        <el-form-item v-if="form.model_cate !== 'cpu'" label="cpu核数">
          <el-input v-model="form.pod_cores">
            <template slot="append"><span class="unit">C</span></template>
          </el-input>
        </el-form-item>
        <el-form-item label="cpu利用率">
          <el-input v-model="form.server_cpu_ratio">
            <template slot="append"><span class="unit">%</span></template>
          </el-input>
        </el-form-item>
        <el-form-item v-show="false" label="平均响应时间">
          <el-input v-model="form.resp_time_avg_all">
            <template slot="append"><span class="unit">ms</span></template>
          </el-input>
        </el-form-item>
        <el-form-item label="内存利用率">
          <el-input v-model="form.server_mem_ratio">
            <template slot="append"><span class="unit">%</span></template>
          </el-input>
        </el-form-item>
        <el-form-item v-if="form.model_cate !== 'throughput'" label="历史配置">
          <el-input v-model="form.pre_alloc" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSubmit">即刻开始推理</el-button>
          <el-button @click="handleReset">重置</el-button>
        </el-form-item>
      </el-form>
      <div v-if="throughputResult && throughputResult.pred_value && form.model_cate === 'throughput'" class="throughputResult">
        <span style="display:inline-block; margin-bottom: 15px;">服务吞吐量：</span><el-tag type="success">{{ throughputResult.pred_value }}</el-tag><br>
        <span>支撑人数：</span><el-tag type="success">{{ (throughputResult.pred_value * 1500 / 920).toFixed(0) }}人</el-tag>
      </div>
    </div>
  </div>
</template>

<script>
import request from '@/utils/request'
import { mapMutations, mapState } from 'vuex'
export default {
  name: 'Documentation',
  components: {},
  data() {
    return {
      form: {
        model_cate: 'cpu',
        people: '1500',
        threads_data_Threads: '33', // 18 ~ 116 +5 循环请求
        server_cpu_ratio: '80',
        resp_time_avg_all: '30', // 平均响应时间 30ms
        server_mem_ratio: '80',
        pod_memory: '2.5', // 内存
        throughput_req: '920', // 吞吐量 2c2.5G
        pre_alloc: '', // 历史配置
        pod_cores: '2' // cpu内核
      },
      throughputResult: null // 吞吐量结果
    }
  },
  computed: {
    ...mapState({
      reasoningResult: state => state.app.reasoningResult
    })
  },
  watch: {
    'form.model_cate': {
      handler(newValue) {
        if (this.form.throughput_req === '920') {
          if (newValue === 'cpu') {
            this.form.pod_memory = '2.5' // 给内存赋默认值
            this.form.pre_alloc = '2'
          } else if (newValue === 'mem') {
            this.form.pod_cores = '2' // 给cpu内核赋默认值
            this.form.pre_alloc = '2.5'
          }
        }
      },
      immediate: true
    }
  },
  methods: {
    ...mapMutations(['app/updateReasoningResult']),
    async handleSubmit() {
      const data_ = Object.assign({}, this.form)
      if (data_.model_cate === 'throughput') { // 吞吐量
        delete data_.throughput_req
        delete data_.pre_alloc
      } else if (data_.model_cate === 'mem') { // 内存
        delete data_.pod_memory
      } else if (data_.model_cate === 'cpu') {
        delete data_.pod_cores
      }
      // const promiseResults = []
      // for (let i = 18; i < 116; i = i + 5) {
      //   const threads_data_Threads = i + ''
      const response = await request({
        url: '/da/infer/',
        method: 'post',
        headers: {
          'Content-Type': 'application/json' // 设置请求的Content-Type
        },
        data: data_ // Object.assign({}, data_, { threads_data_Threads })
      })
      // promiseResults.push({
      //   ...p,
      //   threads_data_Threads
      // })
      // }

      // Promise.all(promiseArrs).then(promiseResults => {
      console.log(response, 'promiseResults')
      // })
      if (data_.model_cate === 'throughput') {
        this.throughputResult = response
      }
      this['app/updateReasoningResult']({ ...response, modelCate: data_.model_cate }) // 缓存推理结果
      // console.log(response, '推理结果')
      this.$message.success('推理完成啦，快去查看结果展示页面吧ヽ(✿ﾟ▽ﾟ)ノ')
    },
    handleReset() {
      this.form = {
        model_cate: 'cpu',
        threads_data_Threads: '33', // 18 ~ 116 +5 循环请求
        server_cpu_ratio: '80',
        resp_time_avg_all: '30', // 平均响应时间 30ms
        server_mem_ratio: '80',
        pod_memory: '2.5', // 内存
        throughput_req: '920', // 吞吐量 2c2.5G
        pre_alloc: '', // 历史配置
        pod_cores: '2' // cpu内核
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.form-container {
  width: 50%;
  margin: 25px auto;
  .throughputResult {
    text-align: center;
  }
  ::v-deep .el-form-item__label {
    font-weight: normal;
  }
  ::v-deep .el-input-group__append .unit {
    display: inline-block;
    width: 20px;
  }
}
</style>
