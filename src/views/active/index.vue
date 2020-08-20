<template>
  <div>
    <!--    <ol-map-active />-->
    <SearchDialog>
      <template v-slot:header>区域活跃度统计</template>
      <template v-slot:center_top>
        <el-button type="primary" plain size="small"> 绘制区域</el-button>
        <el-divider direction="vertical" />
        <el-button type="primary" plain size="small">统计</el-button>
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
// import OlMapActive from '@/views/documentation/components/OlMap'
import SearchDialog from '@/components/SearchDialog/index'
export default {
  name: 'Active',
  components: { SearchDialog },
  data() {
    return {
      textarea: ''
    }
  },
  mounted() {
    // console.log(this.MapContain)
  }
}
</script>

<style scoped>

</style>
