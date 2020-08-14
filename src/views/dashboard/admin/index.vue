<template>
  <div id="app1">
    <OlMap />
  </div>
</template>

<script>
import OlMap from './components/OlMap'
// import DragDialogDemo from '@/views/components-demo/drag-dialog'
export default {
  name: 'DashboardAdmin',
  components: {
    OlMap
  }
}
</script>
<style>
#app1 {
  height: 100%;
}
</style>
