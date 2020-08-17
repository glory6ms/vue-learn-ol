<template>
  <div>
    <ol-map-active />
    <SearchDialog>
      <template v-slot:header>
        <span>轨迹可视化</span>
      </template>
      <template v-slot:center_top>
        <el-date-picker
          v-model="value2"
          type="daterange"
          align="right"
          unlink-panels
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :picker-options="pickerOptions"
          value-format="yyyy-MM-dd"
          style="width: 100%"
        />
      </template>
      <template v-slot:center_bottom>
        <el-button type="primary"> 绘制区域</el-button>
        <el-divider direction="vertical" />
        <el-button type="primary">计算</el-button>
        <el-input
          v-model="textarea"
          type="textarea"
          :autosize="{ minRows: 1, maxRows: 4}"
          placeholder="区域信息"
          style="margin-top: 8px"
        />
      </template>
    </SearchDialog>
  </div>
</template>

<script>
import OlMapActive from '@/views/documentation/components/OlMap'
import SearchDialog from '@/components/SearchDialog/index'
export default {
  name: 'Index',
  components: { SearchDialog, OlMapActive },
  data() {
    return {
      isShow: true, // 查询框体是否折叠
      result_show: false,
      demo: { // 截面经纬度信息及约束条件
        title: '',
        start: '',
        end: ''
      },
      pickerOptions: { // 日期选择器配置
        shortcuts: [{
          text: '2017年2月',
          onClick(picker) {
            const end = new Date('2017-02-07')
            const start = new Date('2017-02-06')
            // start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2018年10月',
          onClick(picker) {
            const end = new Date('2018-10-03')
            const start = new Date('2018-10-01')
            // start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      },
      query: { // axios传参
        line: this.demo,
        date: this.value2
      },
      value1: '',
      value2: '', // 日期选择器的值
      tableData: [],
      textarea: ''
    }
  }
}
</script>

<style scoped>

</style>
