<template>
  <div>
    <ol-map-active />
    <SearchDialog>
      <template v-slot:header>船舶轨迹回放</template>
      <template v-slot:center_top>
        <el-input
          v-model="textarea"
          type="textarea"
          :autosize="{ minRows: 1, maxRows: 4}"
          placeholder="区域边界点经纬度"
          style="margin-top: 8px"
        />
        <el-button type="primary">轨迹回放</el-button>
      </template>
    </SearchDialog>
  </div>

</template>

<script>
import SearchDialog from '@/components/SearchDialog/index'
import OlMapActive from '@/views/documentation/components/OlMap'
export default {
  name: 'ShipTrajectoryReview',
  components: { OlMapActive, SearchDialog },
  data() {
    return {
      textarea: ''
    }
  }
}
</script>

<style scoped>

</style>
