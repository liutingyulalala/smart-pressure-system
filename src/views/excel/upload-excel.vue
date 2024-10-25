<template>
  <div class="app-container">
    <div class="steps">
      <el-steps :active="active" simple style="margin-bottom: 20px">
        <el-step title="上传训练集数据" icon="el-icon-upload" />
        <el-step title="开始训练模型" icon="el-icon-cpu" />
        <el-step title="生成分析报告" icon="el-icon-data-line" />
      </el-steps>
    </div>
    <div class="header">
      <upload-excel-component upload-type="sample" upload-button-text="上传数据" class="uploadButton" :on-success="handleSuccess" :before-upload="beforeUpload" @upload-success="handleUploadSuccess" />
      <el-button class="runButton" :disabled="runButton.disabled" :loading="runButton.loading" type="primary" icon="el-icon-cpu" @click="handleRun">
        {{ runButton.text }}
      </el-button>
      <el-button v-show="websocketEnd" icon="el-icon-view" class="viewReportButton" type="primary" @click="dialogTableVisible = true">
        查看训练模型报告
      </el-button>
    </div>
    <!-- 进度条 -->
    <el-progress v-show="processBtnVisible" class="progress" :text-inside="true" :stroke-width="24" :percentage="process" status="success" />
    <div v-show="!tableDataByPage.length" class="notData">
      <i class="iconfont icon-emptypage" style="font-size: 150px;" />
    </div>
    <!-- 下载链接、图片文本、进度条 ===待办 -->
    <el-table v-show="tableDataByPage.length > 0" :data="tableDataByPage" border highlight-current-row style="width: 100%;margin-top:20px;">
      <el-table-column type="index" />
      <el-table-column v-for="item of tableHeader" :key="item" :prop="item" :label="dataLabelMap[item]">
        <template slot-scope="props">
          {{ props.column.property === 'Time' ? new Date(props.row.Time).toLocaleString() : props.row[props.column.property].toFixed(2) }}
        </template>
      </el-table-column>
    </el-table>
    <Pagination v-show="tableData.length" :total="tableData.length" :page.sync="pageNumber" :limit.sync="pageSize" class="pagination" />
    <!-- 弹框 -->
    <el-dialog title="模型训练报告" fullscreen :visible.sync="dialogTableVisible">
      <!-- 渲染区域 -->
      <div class="response-frame">
        <div class="html-content" v-html="wsData.htmlContents" />
      </div>
    </el-dialog>
  </div>
</template>

<script>
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

export default {
  name: 'UploadExcel',
  components: { UploadExcelComponent, Pagination },
  data() {
    return {
      active: 1,
      dataLabelMap: {
        Time: '时间',
        threads_data_Threads: '模型压力',
        server_cpu_ratio: 'cpu利用率',
        server_mem_ratio: '内存利用率',
        resp_time_avg_all: '平均响应时间',
        pod_cores: 'cpu核数',
        pod_memory: '内存大小',
        throughput_req: '服务吞吐量'
      },
      dialogTableVisible: false,
      runButton: { // 运行按钮
        loading: false,
        disabled: true,
        text: 'Run'
      },
      tableData: [],
      tableHeader: [],
      pageSize: 10,
      pageNumber: 1,
      xlsxFile: null,
      websocket: null,
      wsData: { // ws数据
        rawFile: null,
        htmlContents: ''
      },
      process: 0, // 进度条
      processBtnVisible: false, // 进度条是否显示
      websocketEnd: false, // ws是否结束
      colors: [
        { color: '#f56c6c', percentage: 20 },
        { color: '#e6a23c', percentage: 40 },
        { color: '#5cb87a', percentage: 60 },
        { color: '#1989fa', percentage: 80 },
        { color: '#6f7ad3', percentage: 100 }
      ]
    }
  },
  computed: {
    // 分页表格数据
    tableDataByPage() {
      if (!this.tableData.length) return []
      const res = this.tableData.slice((this.pageNumber - 1) * this.pageSize, this.pageSize * this.pageNumber)
      return res
    }
  },
  methods: {
    handleUploadSuccess(rawFile) {
      this.wsData.rawFile = rawFile
      this.runButton.disabled = false // 当上传文件后，运行按钮才可以点击
    },
    handleImageData(blob) {
      // 将 Blob 直接用作图像数据
      const imageUrl = URL.createObjectURL(blob)
      this.wsData.htmlContents += `<img src="${imageUrl}"/>`
    },
    async handleRun() {
      if (!this.wsData.rawFile) {
        this.$message.info('请上传数据')
        return
      }
      this.active = 2
      this.runButton.text = 'Running'
      this.runButton.loading = true
      this.runButton.disabled = true
      // WebSocket 连接到后端服务
      this.websocket = new WebSocket(`ws://10.2.183.14:18800/ws/train`)

      this.websocket.onopen = () => {
        console.log('WebSocket 连接成功')
        this.processBtnVisible = true
      }

      this.websocket.onmessage = (event) => {
        console.log(event, '发送消息======')
        const logText = event.data
        if (event.data instanceof Blob) {
          // 处理图片数据
          this.handleImageData(event.data)
          return
        } else if (logText.endsWith('Task completed.')) {
          // 判断是否任务完成，如果是则断开 WebSocket 连接
          this.websocket.close()
        } else {
          const logText_ = JSON.parse(logText)[0].split(': ')
          const messageTitle = logText_[0]
          const messageCon = logText_[1]
          if (messageTitle.includes('当前进度')) {
          // 正则表达式匹配百分比数字
            this.process = +(messageCon.split('%')[0])
            console.log(this.process, '进度==========')
          } else if (messageTitle.includes('add_heading')) { // 标题
            this.wsData.htmlContents += `<h2>${messageCon}</h2>`
          } else if (messageTitle.includes('add_paragraph')) { // 段落
            this.wsData.htmlContents += `<p>${messageCon}</p>`
          }
        }
        // 假设最后一个消息是下载链接
        if (logText.startsWith('Download link: ')) {
          this.xlsxFile = logText.replace('Download link: ', '')
        }
      }

      this.websocket.onclose = () => {
        console.log('WebSocket 连接已关闭')
        this.websocketEnd = true
        this.active = 3
        this.runButton.text = 'Run'
        this.runButton.loading = false
        this.runButton.disabled = false
        this.processBtnVisible = false
        this.wsData.rawFile = null
      }

      // 上传文件到 FastAPI 服务器
      const formData = new FormData()
      formData.append('upload_file', this.wsData.rawFile)

      const response = await fetch(`http://10.2.183.14:18800/upload/train`, {
        method: 'POST',
        body: formData
      })

      if (!response.ok) {
        this.runButton.text = 'Run'
        this.runButton.disabled = false
        return
      }
    },
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 1

      if (isLt1M) {
        return true
      }

      this.$message({
        message: 'Please do not upload files larger than 1m in size.',
        type: 'warning'
      })
      return false
    },
    handleSuccess({ results, header }) {
      console.log(header)
      this.tableData = results
      this.tableHeader = header
    }
  }
}
</script>
<style lang="scss" scoped>
   .header {
    display: flex;
    align-items: center;
    .uploadButton {
      margin-right: 15px;
    }
  }
  .notData {
    text-align: center;
    padding: 50px;
    img {
      width: 250px;
    }
    div {
      color: #b2adad;
      font-size: 16px;
      span {
        margin-left: 20px;
      }
    }
  }
  .pagination {
    float: right;
  }
  .progress {
    margin-top: 15px;
  }
  .response-frame {
    background: #eef1f6;
    padding: 8px 24px;
    margin-bottom: 20px;
    border-radius: 2px;
    display: block;
    line-height: 32px;
    font-size: 16px;
    font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
    color: #2c3e50;
    -webkit-font-smoothing: antialiased;
    ::v-deep img {
      width: 45%;
      margin: 0 auto;
      display: block;
    }
  }
</style>
