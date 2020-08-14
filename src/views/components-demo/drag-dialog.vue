<template>
  <div class="components-container">
    <!--    <el-button type="primary" @click="dialogTableVisible = true">-->
    <!--      open a Drag Dialog-->
    <!--    </el-button>-->
    <el-dialog
      v-el-drag-dialog
      :visible.sync="dialogTableVisible"
      title="Shipping"
      :modal="false"
      :close-on-click-modal="false"
      :modal-append-to-body="false"
      :close-on-press-escape="false"
      width="15%"
      @dragDialog="handleDrag"
    >
      <!--      <el-select ref="select" v-model="value" placeholder="请选择">-->
      <!--        <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />-->
      <!--      </el-select>-->
      <!--      <el-table :data="gridData">-->
      <!--        <el-table-column property="mmsi" width="50" />-->
      <!--        <el-table-column property="location" />-->
      <!--        <el-table-column property="time" />-->
      <!--      </el-table>-->
    </el-dialog>
  </div>
</template>

<script>
import elDragDialog from '@/directive/el-drag-dialog'

export default {
  name: 'DragDialogDemo',
  directives: { elDragDialog },
  components: {
  },
  data() {
    return {
      dialogTableVisible: true,
      // options: [
      //   { value: '选项1', label: '黄金糕' },
      //   { value: '选项2', label: '双皮奶' },
      //   { value: '选项3', label: '蚵仔煎' },
      //   { value: '选项4', label: '龙须面' }
      // ],
      // value: '',
      gridData: []
    }
  },
  // watch: {
  //   gridData: function(val) {
  //     this.dialogTableVisible = true
  //     console.log('result')
  //   }
  // },
  methods: {
    // v-el-drag-dialog onDrag callback function
    handleDrag() {
      this.$refs.select.blur()
      // this.griaData = =
    }
  }
}
</script>
<style>
.el-dialog__wrapper {
  position: fixed;
  top: 0;
  right: 30%;
  bottom: 0;
  left: -30%;
  overflow: auto;
  margin: 0;
}
</style>
