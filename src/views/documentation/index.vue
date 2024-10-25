<template>
  <div class="doc-wrapper">
    <!-- <el-alert
      title="提示"
      type="info"
      description="推理完成2秒后将跳转到价值成果页面"
      show-icon
    /> -->
    <el-form
      ref="form"
      :model="form"
      label-width="150px"
      class="form-container"
    >
      <el-form-item label="模型类型">
        <el-radio-group v-model="form.model_cate">
          <el-radio label="cpu">cpu核数</el-radio>
          <el-radio label="throughput">服务吞吐量</el-radio>
          <el-radio label="mem">内存大小</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="模型压力">
        <el-input v-model="form.threads_data_Threads">
          <template slot="append"><span class="unit">个</span></template>
        </el-input>
      </el-form-item>
      <el-form-item label="cpu利用率">
        <el-input v-model="form.server_cpu_ratio">
          <template slot="append"><span class="unit">%</span></template>
        </el-input>
      </el-form-item>
      <el-form-item label="平均响应时间">
        <el-input v-model="form.resp_time_avg_all">
          <template slot="append"><span class="unit">ms</span></template>
        </el-input>
      </el-form-item>
      <el-form-item label="内存利用率">
        <el-input v-model="form.server_mem_ratio">
          <template slot="append"><span class="unit">%</span></template>
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
      <el-form-item v-if="form.model_cate !== 'throughput'" label="服务吞吐量">
        <el-input v-model="form.throughput_req">
          <template slot="append"><span class="unit" style="font-size: 12px;">TPS</span></template>
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
  </div>
</template>

<script>
import request from '@/utils/request'
import { mapMutations } from 'vuex'
export default {
  name: 'Documentation',
  components: {},
  data() {
    return {
      form: {
        model_cate: 'cpu',
        threads_data_Threads: '10.0',
        server_cpu_ratio: '0.8',
        resp_time_avg_all: '12',
        server_mem_ratio: '0.4',
        pod_memory: '8',
        throughput_req: '100',
        pre_alloc: '8',
        pod_cores: ''
      }
    }
  },
  methods: {
    ...mapMutations(['app/updateReasoningResult']),
    async handleSubmit() {
      const data_ = Object.assign({}, this.form)
      if (data_.model_cate === 'throughput') {
        delete data_.throughput_req
        delete data_.pre_alloc
      } else if (data_.model_cate === 'mem') {
        delete data_.pod_memory
      } else if (data_.model_cate === 'cpu') {
        delete data_.pod_cores
      }
      const response = await request({
        url: '/da/infer/',
        method: 'post',
        headers: {
          'Content-Type': 'application/json' // 设置请求的Content-Type
        },
        data: data_
      })
      this['app/updateReasoningResult']({ ...response, modelCate: data_.model_cate }) // 缓存推理结果
      // const response = { 'pred_value': 222, 'diff_value': 268, 'pre_alloc': 490, 'unit_price': 3000 }
      console.log(response, '推理结果')
      this.$message.success('推理完成啦，快去查看结果展示页面吧ヽ(✿ﾟ▽ﾟ)ノ')
    },
    handleReset() {
      this.form = {
        model_cate: 'cpu',
        threads_data_Threads: '',
        server_cpu_ratio: '',
        resp_time_avg_all: '',
        server_mem_ratio: '',
        pod_memory: '',
        throughput_req: '',
        pre_alloc: '',
        pod_cores: ''
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.form-container {
  width: 50%;
  margin: 25px auto;
  ::v-deep .el-form-item__label {
    font-weight: normal;
  }
  ::v-deep .el-input-group__append .unit {
    display: inline-block;
    width: 20px;
  }
}
</style>
