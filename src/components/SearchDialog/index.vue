<template>
  <div id="draggable1" class="box box-1">
    <el-card v-show="isShow" class="box-card">
      <div slot="header" style="text-align: left">
        <slot name="header" />
        <el-button style="float: right; padding: 3px 0" type="text" @click="FunClose"><i class="el-icon-close" /></el-button>
      </div>
      <slot name="center_top" />
      <div style="margin: 10px 0;" />
      <!--      <el-divider><el-tag type="info" effect="plain">设置截面</el-tag></el-divider>-->
      <slot name="center_bottom" />
    </el-card>
    <div>
      <el-button v-show="!isShow" style="float: left" type="success" plain @click="FunClose"><i class="el-icon-zoom-in" /></el-button>
    </div>
  </div>
</template>
<script>
// import * as turf from '@turf/turf'
export default {
  name: 'SearchDialog',
  components: {
  },
  data() {
    return {
      isShow: true // 查询框体是否折叠
    }
  },
  methods: {
    FunClose: function() { // 控制查询框显示
      this.isShow = !this.isShow
    }
  }

}
</script>

<style scoped>
.box {
  position: absolute;
  background: transparent;
  border-radius: 10px;
  left: 1%;
  top: 2%;
  float-displace: auto;
  margin:0 auto;
  opacity: 0.9;
  z-index: 999;
  text-align: center;
  width:  280px;
  display: block;
  font-size: smaller;
}
</style>
