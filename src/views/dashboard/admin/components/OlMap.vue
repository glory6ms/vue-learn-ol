<template>
  <div>
    <div id="map" />
    <QueryBuilderForFlow />
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import XYZ from 'ol/source/XYZ'
import VectorSource from 'ol/source/Vector'
import VectorLayer from 'ol/layer/Vector'
import { Fill, Style, Stroke, Circle } from 'ol/style'
import QueryBuilderForFlow from '@/views/dashboard/admin/components/QueryBuilderForFlow'
import Draw from 'ol/interaction/Draw'
// import { defaults as defaultControls } from 'ol/control'
// import ZoomSlider from 'ol/control/ZoomSlider'

// count用于控制目前所绘制的点为起点还是终点，clickflag用于控制地图单击事件是否会引起文本框的改变
let count = 1
let clickflag = 0
let end_point = new Array(2)
let start_point = new Array(2)
/**
 * source1矢量线段源，保存所绘制截面的端点信息
 * vector1显示绘制线段的图层，需要设置zindex为最上层
 * @type {VectorSource} wrap横轴不重复
 */
const source1 = new VectorSource({ wrapX: false })
let draw1
export default {
  name: 'OlMap',
  components: {
    QueryBuilderForFlow // 子组件为条件查询框
  },
  data() {
    return {
      map: null,
      StartPoint: new Array(2),
      EndPoint: new Array(2)
    }
  },
  mounted() {
    // dom渲染完成后开始初始化地图容器
    this.initMap()
    // this.$parent.$data.map = map;
    this.map.on('singleclick', function(evt) {
      if (clickflag === 1) {
        if (count % 2 === 0) {
          // eslint-disable-next-line no-unused-vars
          end_point = [evt.coordinate[0].toFixed(6), evt.coordinate[1].toFixed(6)]
          console.log('end' + end_point)
        } else {
          // eslint-disable-next-line no-unused-vars
          start_point = [evt.coordinate[0].toFixed(6), evt.coordinate[1].toFixed(6)]
          console.log('start' + start_point)
        }
        count = count + 1
      }
    })
  },
  methods: {
    // 初始化地图
    initMap() {
      // //画断面图层
      const vector1 = new VectorLayer({
        source: source1,
        style: new Style({ // 修改绘制的样式
          fill: new Fill({
            color: 'rgba(255, 255, 255, 0.2)'
          }),
          stroke: new Stroke({
            color: '#ffcc33',
            width: 2
          }),
          image: new Circle({
            radius: 7,
            fill: new Fill({
              color: '#ffcc33'
            })
          })
        })
      })
      const view = new View({
        projection: 'EPSG:4326',
        center: [117.36599976909781, 30.1470299097626],
        zoom: 6
      })
      const layer = new TileLayer({
        source: new XYZ({
          url: 'http://www.google.cn/maps/vt/pb=!1m4!1m3!1i{z}!2i{x}!3i{y}!2m3!1e0!2sm!3i345013117!3m8!2szh-CN!3scn!5e1105!12m4!1e68!2m2!1sset!2sRoadmap!4e0'
        })
      })
      this.map = new Map({
        layers: [layer, vector1],
        target: 'map',
        view: view
        // controls: defaultControls().extend([new ZoomSlider()])
      })
    },
    // 绘制截面函数
    draw_line() {
      console.log('请绘制截面')
      clickflag = 1
      this.map.removeInteraction(draw1)
      draw1 = new Draw({
        source: source1,
        type: 'LineString',
        maxPoints: 2
      })
      // 开始绘制
      draw1.on('drawstart', function() {
        source1.clear()
      })
      draw1.on('drawend', function() {
        // 截面线段绘制完成
        // clickflag = 0
        // this.map.removeInteraction(draw1)
      })
      this.map.addInteraction(draw1)
    },
    draw_end() {
      clickflag = 0
      this.map.removeInteraction(draw1)
      this.EndPoint = end_point
      this.StartPoint = start_point
    }
  }
}
</script>

<style scoped>
  #map {
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      width: 100%;
      height: 100%;
  }
</style>
