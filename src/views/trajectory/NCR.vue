<template>
  <div>
    <SearchDialog>
      <template v-slot:header>
        <span>核密度图</span>
      </template>
      <template v-slot:center_top>
        <el-button type="primary" plain> 绘制区域</el-button>
        <el-divider direction="vertical" />
        <el-button type="primary" plain>计算</el-button>
        <el-input
          v-model="textarea"
          type="textarea"
          :autosize="{ minRows: 1, maxRows: 4}"
          placeholder="区域边界点经纬度"
          style="margin-top: 8px"
        />
      </template>
    </SearchDialog>
  </div>

</template>

<script>
import SearchDialog from '@/components/SearchDialog/index'
export default {
  name: 'NCR',
  components: { SearchDialog },
  data() {
    return {
      isShow: true, // 查询框体是否折叠
      pickerOptions: { // 日期选择器配置
        shortcuts: [{
          text: '2017年2月',
          onClick(picker) {
            const end = new Date('2017-02-07')
            const start = new Date('2017-02-06')
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2018年10月',
          onClick(picker) {
            const end = new Date('2018-10-03')
            const start = new Date('2018-10-01')
            picker.$emit('pick', [start, end])
          }
        }]
      },
      value2: '', // 日期选择器的值
      tableData: [],
      textarea: ''
    }
  }
}
</script>

<style scoped>

</style>
