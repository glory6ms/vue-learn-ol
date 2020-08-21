<template>
  <div id="app">
    <div id="map" />
    <div id="mouse-position" />
    <div id="tools">
      <el-button type="primary" class="el-icon-location-information" title="经纬度定位" circle />
      <el-button type="success" class="el-icon-s-flag" title="海图标绘" circle />
      <el-button type="info" class="el-icon-s-promotion" title="距离测量" circle />
      <el-button type="warning" class="el-icon-crop" title="海图范围设置" circle />
      <el-button type="danger" class="el-icon-guide" title="经纬度换算" circle />
    </div>
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
// import XYZ from 'ol/source/XYZ'
import { defaults as defaultControls, MousePosition } from 'ol/control'
import { toStringHDMS } from 'ol/coordinate'
import BingMaps from 'ol/source/BingMaps'
// import ZoomSlider from 'ol/control/ZoomSlider'
export default {
  name: 'OlMapActive',
  components: {
  },
  data() {
    return {
      map: null
    }
  },
  mounted() {
    // dom渲染完成后开始初始化地图容器
    this.initMap()
  },
  methods: {
    // 初始化地图
    initMap() {
      const mousePositionControl = new MousePosition({
        coordinateFormat: function(coord) {
          return toStringHDMS(coord)
        }, // 将坐标保留4位小数位，并转换为字符串
        projection: 'EPSG:4326', // 定义投影
        className: 'custom-mouse-position', // 控件的CSS类名
        target: document.getElementById('mouse-position'), // 将控件渲染在该DOM元素中
        undefinedHTML: '&nbsp;' // 鼠标离开地图时，显示空格
      })
      const view = new View({
        projection: 'EPSG:4326',
        center: [117.365999, 30.1470299],
        zoom: 6
      })
      // const layer = new TileLayer({
      //   source: new XYZ({
      //     url: 'http://www.google.cn/maps/vt/pb=!1m4!1m3!1i{z}!2i{x}!3i{y}!2m3!1e0!2sm!3i345013117!3m8!2szh-CN!3scn!5e1105!12m4!1e68!2m2!1sset!2sRoadmap!4e0'
      //   })
      // })
      const bingMapLayer = new TileLayer({
        source: new BingMaps({
          key: 'AkjzA7OhS4MIBjutL21bkAop7dc41HSE0CNTR5c6HJy8JKc7U9U9RveWJrylD3XJ',
          imagerySet: 'Road'
        })
      })
      this.map = new Map({
        layers: [bingMapLayer],
        target: 'map',
        view: view,
        controls: defaultControls({ attribution: false, zoom: false, rotate: false }).extend([mousePositionControl])
      })
    }
  }
}
</script>

<style scoped>
  #map {
    position: absolute;
    height: 100%;
    right: 5px;
    bottom: 5px;
    top: 5px;
    left: 5px;
  }
  #mouse-position{
    position: absolute;
    right: 5px;
    bottom: 5px;
    z-index: 999;
  }
  #tools{
    position: absolute;
    right: 5px;
    top: 5px;
    z-index: 999;
  }
</style>
