<template>
  <div>
    <head>
      <link
        rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css"
      >
    </head>
    <div class="main-page">
      <div class="upload-section">
        <button class="custom-file-upload" @click="triggerFileInput">
          <i class="fa-solid fa-cloud-arrow-up" />
          <span class="span-upload">Upload Files</span>
        </button>
        <p class="file-format-hint">
          Our application supports input formats including PDF, Word documents
          (DOC, DOCX), and images (JPEG, PNG, etc.).
        </p>
        <input
          id="file-upload"
          ref="fileInput"
          type="file"
          accept=".pdf,.doc,.docx,.jpg,.jpeg,.png,.bmp,.tiff"
          @change="handleFileChange"
        >
      </div>
      <div class="download-section">
        <a v-if="xlsxFile" :href="xlsxFile" download="result.xlsx">
          <button class="btn">
            <i class="fa-solid fa-download" />
            <span>Download Excel</span>
          </button>
        </a>
      </div>
      <div v-if="file" class="detail-content">
        <div class="left-pannel">
          <div ref="logFrame" class="log-frame">
            <h2>File Details:</h2>
            <p>Name: {{ file.name }}</p>
            <p>Size: {{ (file.size / 1024).toFixed(2) }}Kb</p>
            <p>Type: {{ file.type }}</p>
          </div>
          <div class="grogress-frame">
            <label
              style="font-weight: bold"
              for="layout-file"
            >Layout Progress:</label>
            <div class="progress-container">
              <div
                class="progress-bar"
                :style="{ width: layout_process + '%' }"
              >
                <div class="progress-text">{{ layout_process }}%</div>
              </div>
            </div>
            <label
              style="font-weight: bold"
              for="recover-file"
            >Recover Progress:</label>
            <div class="progress-container">
              <div
                class="progress-bar"
                :style="{ width: recover_process + '%' }"
              >
                <div class="progress-text">{{ recover_process }}%</div>
              </div>
            </div>
          </div>

          <button
            v-if="file"
            class="run-button"
            :disabled="isRunning"
            @click="runTask"
          >
            {{ buttonText }}
          </button>
        </div>
        <div class="rigth-pannel">
          <div v-if="combinedContents.length" class="response-frame">
            <div
              v-for="(content, index) in combinedContents"
              :key="index"
              class="content-pair"
            >
              <img
                v-if="content.imageUrl"
                :src="content.imageUrl"
                alt="Received Image"
              >
              <div class="html-content" v-html="content.htmlContent" />
            </div>
          </div>
        </div>
      </div>
      <div v-if="!file" class="intro-text">
        <p>
          Here are individual points describing table extraction technology:
        </p>
        <ul>
          <li>
            <strong>Fast Extraction Speed: </strong>Rapid processing of large
            tables with optimized algorithms ensures quick and efficient data
            handling.
          </li>
          <li>
            <strong>High OCR Accuracy: </strong> Advanced OCR techniques deliver
            precise text and number recognition, minimizing errors and ensuring
            data reliability.
          </li>
          <li>
            <strong>Visualization Support: </strong> Offers intuitive
            visualization options for formatting, filtering, and sorting
            extracted tables, enhancing data analysis.
          </li>
          <li>
            <strong>Accurate Table Line Recognition: </strong>Preserves table
            structures, including lines and borders, maintaining the original
            layout for accurate data presentation.
          </li>
        </ul>
        <p>
          Each of these features contributes to a robust and efficient table
          extraction process, enhancing the accuracy and usability of extracted
          data while streamlining workflows and improving overall productivity.
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
export default defineComponent({
  name: 'UploadPage',
  data() {
    return {
      logs: [],
      xlsxFile: null,
      layout_process: 0,
      recover_process: 0,
      buttonText: 'Run',
      isRunning: false,
      htmlContents: [],
      imageUrls: [],
      apiUrl: import.meta.env.VITE_APP_EXTRACT_TABLES_API_URL,
      file: null,
      websocket: null,
      websocketOpen: false // 初始化为未连接状态
    }
  },
  computed: {
    combinedContents() {
      // 获取两个数组的最短长度
      const minLength = Math.min(
        this.htmlContents.length,
        this.imageUrls.length
      )
      // 生成包含 HTML 内容和图像 URL 的对象数组
      return Array.from({ length: minLength }, (_, index) => ({
        htmlContent: this.htmlContents[index],
        imageUrl: this.imageUrls[index]
      }))
    }
  },
  methods: {
    triggerFileInput() {
      // 触发文件选择对话框
      this.$refs.fileInput.click()
      this.file = null
      this.xlsxFile = null
      this.layout_process = 0
      this.recover_process = 0
      this.htmlContents = []
      this.imageUrls = []
    },
    handleFileChange(event) {
      // 允许的文件类型
      const allowedFormats = [
        'application/pdf', // PDF
        'application/msword', // .doc
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document', // .docx
        'image/jpeg', // .jpg, .jpeg
        'image/png', // .png
        'image/bmp', // .bmp
        'image/tiff' // .tiff
      ]

      const rawFile = event.target.files[0] // 获取上传的文件
      let errorMessage = ''
      // 验证文件类型是否在允许列表中
      if (allowedFormats.includes(rawFile.type)) {
        this.file = rawFile // 如果文件类型合法，则保存文件
      } else {
        // 如果文件类型不合法，则提示错误消息
        errorMessage = `Invalid file type: ${rawFile.name}. Please upload PDF, Word, or image file formats.`
        alert(errorMessage)
      }
    },
    async handleImageData(blob) {
      // 将 Blob 直接用作图像数据
      const imageUrl = URL.createObjectURL(blob)
      this.imageUrls.push(imageUrl)
    },
    generateRandomHexId(length = 8) {
      let result = ''
      for (let i = 0; i < length; i++) {
        result += Math.floor(Math.random() * 16).toString(16)
      }
      return result
    },
    async runTask() {
      if (!this.file) return
      this.buttonText = 'Running'
      this.isRunning = true
      const clientId = this.generateRandomHexId()
      // WebSocket 连接到后端服务
      this.websocket = new WebSocket(`ws://${this.apiUrl}/ws/${clientId}`)

      this.websocket.onopen = () => {
        console.log('WebSocket 连接成功')
        this.logs.push('WebSocket 连接成功')
        this.websocketOpen = true // 连接成功后禁用按钮
      }

      this.websocket.onmessage = (event) => {
        const logText = event.data
        if (event.data instanceof Blob) {
          // 处理图片数据
          this.handleImageData(event.data)
          this.logs.push(logText)
          return
        }
        if (logText.includes('Layout Progress')) {
          // 正则表达式匹配百分比数字
          const regex = /\((\d+\.\d+)%\)/
          const match = logText.match(regex)
          if (match) {
            this.layout_process = Math.floor(parseFloat(match[1]))
          }
        } else if (logText.includes('Recover Progress')) {
          const regex = /\((\d+\.\d+)%\)/
          const match = logText.match(regex)
          if (match) {
            this.recover_process = Math.floor(parseFloat(match[1]))
          }
        } else if (logText.includes('html content: ')) {
          this.htmlContents.push(logText.replace('html content: ', ''))
        }
        this.logs.push(logText)

        // 判断是否任务完成，如果是则断开 WebSocket 连接
        if (logText.endsWith('Task completed.')) {
          this.websocket.close()
        }

        // 假设最后一个消息是下载链接
        if (logText.startsWith('Download link: ')) {
          this.xlsxFile = logText.replace('Download link: ', '')
        }
      }

      this.websocket.onclose = () => {
        console.log('WebSocket 连接已关闭')
        this.logs.push('WebSocket 连接已关闭')
        this.websocketOpen = false // 连接关闭后恢复按钮
        this.buttonText = 'Run'
        this.isRunning = false
      }

      // 上传文件到 FastAPI 服务器
      const formData = new FormData()
      formData.append('upload_file', this.file)

      const response = await fetch(`http://${this.apiUrl}/upload/${clientId}`, {
        method: 'POST',
        body: formData
      })

      if (!response.ok) {
        console.error('文件上传失败')
        this.logs.push('文件上传失败')
        this.buttonText = 'Run'
        this.isRunning = false
        return
      } else {
        console.log('文件上传成功')
        this.logs.push('文件上传成功')
      }
    }
  }
})
</script>

<style scoped>
.main-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 40px;
}

.upload-section {
  display: flex;
  width: 100%;
  flex-direction: column;
  align-items: center;
}

.left-pannel {
  display: flex;
  max-width: calc(30% - 10px);
  text-align: left;
  flex-direction: column;
  align-items: center;

  width: 100%;
}
#file-upload {
  display: none;
  /* 隐藏实际的文件输入框 */
}
.download-section {
  width: 100%;
  display: flex;
  justify-content: flex-end;
}

.rigth-pannel {
  flex: 1;
  margin-bottom: 20px;
  text-align: center;
  max-width: calc(70% - 10px);
  margin-left: 20px;
}

.detail-content {
  display: flex;
  width: 100%;
}

.grogress-frame {
  display: flex;
  flex-direction: column;
  width: 100%;
  margin-top: 10px;
}

.process {
  width: 100%;
}

.log-frame {
  flex: 1;
  height: 300px;
  /* 设置固定高度 */
  overflow-y: auto;
  /* 如果日志超过框架高度，显示滚动条 */
  border: 1px solid #ccc;
  padding-left: 10px;
  width: 100%;
}

.run-button {
  display: flex;
  margin-top: 10px;
}

.response-frame {
  max-height: 60vh;
  /* 设置最大高度 */
  overflow-y: auto;
  /* 设置垂直滚动条 */
  border: 1px solid #ccc;
  /* 可选：添加边框 */
  padding: 10px;
  /* 可选：添加内边距 */
  display: flex;
  /* 使图片和文字并排显示 */
  flex-direction: column;
  /* 使每对图片和文本垂直排列 */
}

.btn {
  background-color: #8697c4;
  border: none;
  color: white;
  padding: 8px 12px;
  cursor: pointer;
  font-size: 16x;
  margin-right: 10px;
  margin-bottom: 5px;
}

.btn i {
  margin-right: 4px;
}

/* Darker background on mouse-over */
.btn:hover {
  background-color: RoyalBlue;
}

.custom-file-upload {
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  background-color: #8697c4;
  cursor: pointer;
  transition: background-color 0.3s;
  color: white;
  margin-top: 10px;
  margin-bottom: 0px;
  width: 30%;
}

.content-pair {
  display: flex;
}

.content-pair img {
  margin-right: 10px;
  max-width: 800px;
  object-fit: contain;
}
.content-pair div {
  min-width: 1000px;
  overflow: auto;
  flex: 1 1 auto; /* 使内容在容器中自适应 */
  font-size: 0.5em; /* 调整字体大小为默认的80% */
  text-align: left; /* 文本左对齐 */
  white-space: nowrap; /* 防止换行 */
  align-content: center;
}
.custom-file-upload:hover {
  background-color: RoyalBlue;
}

.custom-file-upload i {
  font-size: 24px;
  /* 图标大小 */
  margin-right: 8px;
  /* 图标和文本之间的间距 */
}

.custom-file-upload .span-upload {
  font-size: 16px;
  /* 文本大小 */
}

.file-format-hint {
  font-size: 13px;
  /* 设置提示字体大小 */
  color: #888;
  /* 设置提示字体颜色为浅灰色 */
  display: block;
  /* 显示为块级元素 */
  margin-top: 1px;
  /* 增加上方间距 */
}

.intro-text {
  text-align: left;
  line-height: 1.6;
  color: #333;
  max-width: 800px;
  margin-top: 20px;
  padding: 20px;
}

.intro-text ul {
  margin-bottom: 10px;
}

.intro-text li {
  margin-bottom: 8px;
}

.intro-text strong {
  color: #007bff;
  /* Blue color for strong tags */
}

button {
  margin-top: 10px;
  /* 添加顶部边距 */
  padding: 10px 20px;
  /* 添加内边距 */
  background-color: #8697c4;
  /* 背景颜色 */
  color: white;
  /* 字体颜色 */
  border: none;
  /* 去除边框 */
  border-radius: 5px;
  /* 添加圆角 */
  cursor: pointer;
  /* 鼠标指针 */
}

button:hover {
  background-color: RoyalBlue;
  /* 悬停背景颜色 */
}

button:disabled {
  background-color: #cccccc;
  /* 禁用背景颜色 */
  cursor: not-allowed;
  /* 禁用鼠标指针 */
}

.progress-container {
  width: 100%;
  background-color: #f3f3f3;
  border: 1px solid #ccc;
  border-radius: 5px;
  overflow: hidden;
  position: relative;
  height: 30px;
  /* Ensure this matches the height of progress-bar */
  margin-bottom: 10px;
  margin-top: 5px;
}

.progress-bar {
  height: 100%;
  background-color: #3d52a0;
  position: relative;
  transition: width 0.4s ease;
}

.progress-text {
  position: absolute;
  width: 100%;
  text-align: center;
  line-height: 30px;
  /* Match this to the height of progress-bar */
  color: white;
  font-weight: bold;
  top: 0;
  left: 0;
}
</style>
