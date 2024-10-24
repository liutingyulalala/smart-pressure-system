<template>
  <div class="doc-wrapper">
    <el-alert
      title="提示"
      type="info"
      description="推理完成2秒后将跳转到价值成果页面"
      show-icon
    />
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
        <el-input v-model="form.threads_data_Threads" />
      </el-form-item>
      <el-form-item label="cpu利用率">
        <el-input v-model="form.server_cpu_ratio" />
      </el-form-item>
      <el-form-item label="平均响应时间">
        <el-input v-model="form.resp_time_avg_all" />
      </el-form-item>
      <el-form-item label="内存利用率">
        <el-input v-model="form.server_mem_ratio" />
      </el-form-item>
      <el-form-item v-if="form.model_cate !== 'mem'" label="内存大小">
        <el-input v-model="form.pod_memory" />
      </el-form-item>
      <el-form-item v-if="form.model_cate !== 'cpu'" label="cpu核数">
        <el-input v-model="form.pod_cores" />
      </el-form-item>
      <el-form-item v-if="form.model_cate !== 'throughput'" label="服务吞吐量">
        <el-input v-model="form.throughput_req" />
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
export default {
  name: 'Documentation',
  components: {},
  data() {
    return {
      form: {
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
  },
  methods: {
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
      // const response = { 'pred_value': 222, 'diff_value': 268, 'pre_alloc': 490, 'unit_price': 3000 }
      console.log(response, '推理结果')
    },
    handleReset() {
      this.form = {
        model_cate: '',
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
  width: 40%;
  margin: 25px auto;
  ::v-deep .el-form-item__label {
    font-weight: normal;
  }
}
</style>
