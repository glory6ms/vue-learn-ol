<template>
  <div>
    <OlMap />
  </div>
</template>

<script>
import OlMap from './components/OlMap'
export default {
  name: 'DashboardAdmin',
  components: {
    OlMap
  },
  data() {
    return {
      map: null
    }
  }
}
</script>
