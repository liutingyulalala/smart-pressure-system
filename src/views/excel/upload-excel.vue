<template>
  <div class="app-container">
    <upload-excel-component :on-success="handleSuccess" :before-upload="beforeUpload" />
    <el-table v-show="tableDataByPage.length > 0" :data="tableDataByPage" border highlight-current-row style="width: 100%;margin-top:20px;">
      <el-table-column type="index" />
      <el-table-column v-for="item of tableHeader" :key="item" :prop="item" :label="item" />
    </el-table>
    <Pagination v-show="tableData.length" :total="tableData.length" :page.sync="pageNumber" :limit.sync="pageSize" />
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
      tableData: [],
      tableHeader: [],
      pageSize: 10,
      pageNumber: 1
    }
  },
  computed: {
    // 分页表格数据
    tableDataByPage() {
      console.log(this.pageNumber, 'pageNumber')
      if (!this.tableData.length) return []
      const res = this.tableData.slice((this.pageNumber - 1) * this.pageSize, this.pageSize * this.pageNumber)
      console.log(res, 'res')
      return res
    }
  },
  methods: {
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
      console.log(results)
      this.tableData = results
      this.tableHeader = header
    }
  }
}
</script>
