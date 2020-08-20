<template>
  <section class="app-main">
    <transition name="fade-transform" mode="out-in">
      <keep-alive :include="cachedViews">
        <div>
          <ol-map-active v-show="isShow" ref="myMap" />
          <router-view :key="key" :map="map" />
        </div>
      </keep-alive>
    </transition>
  </section>
</template>

<script>
import OlMapActive from '@/views/documentation/components/OlMap'
export default {
  name: 'AppMain',
  components: { OlMapActive },
  data() {
    return {
      map: ''
    }
  },
  computed: {
    cachedViews() {
      return this.$store.state.tagsView.cachedViews
    },
    key() {
      return this.$route.path
    },
    isShow() {
      return !(this.$route.path === '/charts/index')
    }
  },
  mounted() {
    this.map = this.$refs.myMap.map
  }
}
</script>

<style lang="scss" scoped>
.app-main {
  /* 50= navbar  50  */
  min-height: 90%;
  width: 100%;
  position: relative;
  overflow: hidden;
}

.fixed-header+.app-main {
  padding-top: 50px;
}

.hasTagsView {
  .app-main {
    /* 84 = navbar + tags-view = 50 + 34 */
    min-height: calc(100vh - 84px);
  }

  .fixed-header+.app-main {
    padding-top: 84px;
  }
}
</style>

<style lang="scss">
// fix css style bug in open el-dialog
.el-popup-parent--hidden {
  .fixed-header {
    padding-right: 15px;
  }
}
</style>
