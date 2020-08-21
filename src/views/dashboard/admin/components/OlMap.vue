<template>
  <div id="app">
    <div id="map" />
    <div id="mouse-position" />
    <div id="tools">
      <i class="el-icon-location-information" title="经纬度定位" />
      <i class="el-icon-s-flag" title="海图标绘" />
      <i class="el-icon-s-promotion" title="距离测量" />
      <i class="el-icon-crop" title="海图范围设置" />
      <i class="el-icon-guide" title="经纬度换算" />
    </div>
    <QueryBuilderForFlow />
  </div>
</template>

<script>
import { Message } from 'element-ui'
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
// import XYZ from 'ol/source/XYZ'
import VectorSource from 'ol/source/Vector'
import VectorLayer from 'ol/layer/Vector'
import { Fill, Style, Stroke, Circle } from 'ol/style'
import QueryBuilderForFlow from '@/views/dashboard/admin/components/QueryBuilderForFlow'
import Draw from 'ol/interaction/Draw'
import { defaults as defaultControls, MousePosition } from 'ol/control'
import { toStringHDMS } from 'ol/coordinate'
import Feature from 'ol/Feature'
import WebGLPointsLayer from 'ol/layer/WebGLPoints'
import Point from 'ol/geom/Point'
import BingMaps from 'ol/source/BingMaps'
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
const PointVectorSource = new VectorSource({})
let pointsLayer
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
          end_point = [evt.coordinate[0].toFixed(4), evt.coordinate[1].toFixed(4)]
          // this.EndPoint = end_point
          console.log('end:' + end_point)
          clickflag = 0
          this.open_msg('warning', '请点击流量统计按钮')
        } else {
          // eslint-disable-next-line no-unused-vars
          start_point = [evt.coordinate[0].toFixed(4), evt.coordinate[1].toFixed(4)]
          // this.StartPoint = start_point
          console.log('start:' + start_point)
        }
        count = count + 1
      }
    })
  },
  methods: {
    // 初始化地图
    initMap() {
      // 创建Mouseposition控件
      const mousePositionControl = new MousePosition({
        coordinateFormat: function(coord) {
          return toStringHDMS(coord)
        }, // 将坐标保留4位小数位，并转换为字符串
        projection: 'EPSG:4326', // 定义投影
        className: 'custom-mouse-position', // 控件的CSS类名
        target: document.getElementById('mouse-position'), // 将控件渲染在该DOM元素中
        undefinedHTML: '&nbsp;' // 鼠标离开地图时，显示空格
      })
      // //画断面图层
      const vector1 = new VectorLayer({
        source: source1,
        style: new Style({ // 修改绘制的样式
          fill: new Fill({
            color: 'rgba(255,255,255,0.2)'
          }),
          stroke: new Stroke({
            color: 'blue',
            width: 2
          }),
          image: new Circle({
            radius: 7,
            fill: new Fill({
              color: 'blue'
            })
          })
        }),
        zIndex: 999
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
      var bingMapLayer = new TileLayer({
        source: new BingMaps({
          key: 'AkjzA7OhS4MIBjutL21bkAop7dc41HSE0CNTR5c6HJy8JKc7U9U9RveWJrylD3XJ',
          imagerySet: 'Road'
        })
      })
      this.map = new Map({
        layers: [bingMapLayer, vector1],
        target: 'map',
        view: view,
        controls: defaultControls({ attribution: false, zoom: false, rotate: false }).extend([mousePositionControl])
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
      const that = this
      draw1.on('drawend', function() {
        // 截面线段绘制完成
        // clickflag = 0
        that.map.removeInteraction(draw1)
      })
      this.map.addInteraction(draw1)
    },
    draw_end() {
      // this.map.removeInteraction(draw1)
      this.EndPoint = end_point
      this.StartPoint = start_point
    },
    point_layer(list) {
      if (list.length === 0) {
        alert('此时间段查询不到数据')
        return
      }
      const features = []
      list.forEach(function(v) {
        const location = v.content.location
        const feature = new Feature({
          geometry: new Point(location)
        })
        features.push(feature)
      })
      PointVectorSource.clear()
      PointVectorSource.addFeatures(features)
      const predefinedStyles = {
        'circles': {
          symbol: {
            symbolType: 'circle',
            size: [
              'interpolate',
              ['linear'],
              ['get', 'population'],
              40000,
              8,
              2000000,
              28
            ],
            color: '#fe2020',
            rotateWithView: false,
            offset: [0, 0],
            opacity: [
              'interpolate',
              ['linear'],
              ['get', 'population'],
              40000,
              0.6,
              2000000,
              0.92
            ]
          }
        },
        'circles-zoom': {
          symbol: {
            symbolType: 'circle',
            size: ['interpolate', ['exponential', 2.5], ['zoom'], 2, 1, 14, 32],
            color: '#00aa17',
            offset: [0, 0],
            opacity: 0.95
          }
        }
      }
      const previousLayer = pointsLayer
      pointsLayer = new WebGLPointsLayer({
        source: PointVectorSource,
        style: predefinedStyles['circles'],
        disableHitDetection: true
      })
      this.map.addLayer(pointsLayer)
      if (previousLayer) {
        this.map.removeLayer(previousLayer)
        previousLayer.dispose()
      }
      this.map.getView().fit(PointVectorSource.getExtent(), this.map.getSize())
      // quz???这几行代码干嘛用得
      // function animate() {
      //   this.map.render()
      //   window.requestAnimationFrame(animate)
      // }
      // animate()
    },
    open_msg: function(type, msg) {
      Message({
        message: msg,
        type: type,
        duration: 5000
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
    top: 2px;
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
