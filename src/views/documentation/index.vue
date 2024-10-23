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
          <el-radio label="throughput">吞吐量</el-radio>
          <el-radio label="mem">内存大小</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="模型压力">
        <el-input v-model="form.threads_data_Threads" />
      </el-form-item>
      <el-form-item label="cpu利用率">
        <el-input v-model="form.server_cpu_ratio" />
      </el-form-item>
      <el-form-item label="内存利用率">
        <el-input v-model="form.server_mem_ratio" />
      </el-form-item>
      <el-form-item label="平均响应时间">
        <el-input v-model="form.resp_time_avg_all" />
      </el-form-item>
      <el-form-item label="内存大小">
        <el-input v-model="form.pod_memory" />
      </el-form-item>
      <el-form-item label="服务的吞吐量">
        <el-input v-model="form.throughput_req" />
      </el-form-item>
      <el-form-item label="历史配置">
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
        threads_data_Threads: 12.5,
        server_cpu_ratio: 0.8,
        resp_time_avg_all: 33.8,
        server_mem_ratio: 0.6,
        pod_memory: 8.0,
        throughput_req: 100.0,
        pre_alloc: 3
      }
    }
  },
  methods: {
    async handleSubmit() {
      console.log(this.form)

      // const response = fetch('/dev-api/da/infer/', {
      //   method: 'POST', // 指定请求方法
      //   headers: {
      //     'Content-Type': 'application/json' // 设置请求的Content-Type
      //   },
      //   body: JSON.stringify(this.form) // 将数据转换为JSON字符串作为请求体
      // })
      const response = await request({
        url: '/da/infer/',
        method: 'post',
        headers: {
          'Content-Type': 'application/json' // 设置请求的Content-Type
        },
        data: this.form
      })
      // const response = { 'pred_value': 222, 'diff_value': 268, 'pre_alloc': 490, 'unit_price': 3000 }
      console.log(response, '推理结果')
    },
    handleReset() {
      this.form = {
        model_cate: '',
        threads_data_Threads: 12.5,
        server_cpu_ratio: 0.8,
        resp_time_avg_all: 33.8,
        server_mem_ratio: 0.6,
        pod_memory: 8.0,
        throughput_req: 100.0,
        pre_alloc: 3
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
