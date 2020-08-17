<template>
  <div style="height: 100%;">
    <ol-map-active />
    <SearchDialog>
      <template v-slot:header>区域活跃度统计</template>
      <template v-slot:center_top>
        <el-row :gutter="20">
          <el-col :span="16"><el-button type="primary"> 绘制多边形区域</el-button></el-col>
          <el-col :span="8"><el-button type="info">统计</el-button></el-col>
        </el-row>
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
import OlMapActive from '@/views/documentation/components/OlMap'
import SearchDialog from '@/components/SearchDialog/index'
export default {
  name: 'Active',
  components: { SearchDialog, OlMapActive },
  data() {
    return {
      textarea: ''
    }
  }
}
</script>

<style scoped>

</style>
