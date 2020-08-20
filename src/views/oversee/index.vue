<template>
  <div>
    <!--    <ol-map-active />-->
    <SearchDialog>
      <template v-slot:header>
        <span>监控船舶与区域</span>
      </template>
      <template v-slot:center_top>
        <el-divider content-position="left">监控</el-divider>
        <el-input v-model="input_MMSI" placeholder="船舶mmsi" class="input-with-select" type="text">
          <!--          <template slot="prepend">监控</template>-->
          <!--          <span slot="prefix">监控</span>-->
          <el-button slot="append" icon="el-icon-search" />
        </el-input>
      </template>
      <template v-slot:center_bottom>
        <el-row :gutter="20">
          <el-col :span="16"><el-button type="primary" plain> 绘制监控区域</el-button></el-col>
          <el-col :span="8"><el-button type="primary" plain>监控</el-button></el-col>
        </el-row>
        <el-input
          v-model="textarea"
          type="textarea"
          :autosize="{ minRows: 1, maxRows: 4}"
          placeholder="监控区域信息"
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
  name: 'Index',
  components: { SearchDialog },
  data: function() {
    return {
      input_MMSI: '',
      textarea: ''
    }
  }
}
</script>

<style scoped>

</style>
