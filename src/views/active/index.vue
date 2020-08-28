<template>
  <div>
    <!--    <ol-map-active />-->
    <SearchDialog v-loading="loading">
      <template v-slot:header>区域活跃度统计</template>
      <template v-slot:center_top>
        <el-date-picker
          v-model="value2"
          type="daterange"
          align="right"
          unlink-panels
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :picker-options="pickerOptions"
          value-format="yyyy-MM-dd"
          style="width: 100%"
          @change="Query_Trajectory"
        />
      </template>
      <template v-slot:center_bottom>
        <el-button type="primary" plain size="small" @click="drawPoly"> 绘制区域</el-button>
        <el-divider direction="vertical" />
        <el-button type="primary" plain size="small" @click="active_count">统计</el-button>
        <el-input
          v-model="textarea"
          type="textarea"
          :autosize="{ minRows: 1, maxRows: 4}"
          placeholder="区域边界点经纬度"
          style="margin-top: 8px"
        />
      </template>
    </SearchDialog>
  </div>
</template>

<script>
import * as turf from '@turf/turf'
import SearchDialog from '@/components/SearchDialog/index'
import { Notification } from 'element-ui'
import VectorSource from 'ol/source/Vector'
import VectorLayer from 'ol/layer/Vector'
import { Circle, Fill, Stroke, Style } from 'ol/style'
import axios from 'axios'
import Feature from 'ol/Feature'
import Point from 'ol/geom/Point'
import WebGLPointsLayer from 'ol/layer/WebGLPoints'
import Draw from 'ol/interaction/Draw'
import Format from 'ol/format/GeoJSON'
export default {
  name: 'Active',
  components: { SearchDialog },
  props: ['map'],
  data() {
    return {
      textarea: '',
      result_show: false,
      pickerOptions: { // 日期选择器配置
        shortcuts: [{
          text: '2018年10月',
          onClick(picker) {
            const end = new Date('2018-10-03')
            const start = new Date('2018-10-01')
            // start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      },
      value2: '', // 日期选择器的值
      buffer_area: { // 截面经纬度信息及约束条件
        in: [],
        out: []
      },
      loading: false
    }
  },
  computed: {
  },
  mounted() {
  },
  methods: {
    Query_Trajectory: function() {
      if (this.value2 === '') {
        Notification.error('请填写查询时间段')
        return
      }
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1]
      }
      this.loading = true
      const that = this
      axios.post('http://localhost:5003/queryByTime', params)
        .then(function(response) {
          that.loading = false
          if (response.data.length === 0) {
            Notification.warning('查询结果数为零')
            return
          }
          that.Point_Layer(response.data)
          Notification.success('查询成功!')
        })
        .catch(function(error) {
          that.loading = false
          Notification.error('查询失败，请重新输入时间段')
          console.log(error)
        })
    },
    active_count() {
      // const arr = JSON.parse(this.textarea)
      // const start_point = arr[0][0]
      // const end_point = arr[0][2]
      if (this.textarea === '' || this.value2 === '') {
        Notification.error('请选择时间与区域！')
        return
      }
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1],
        in: this.buffer_area.in,
        out: this.buffer_area.out
      }
      this.loading = true
      const that = this
      axios.post('http://localhost:5003/QueryByPolygonRange', params)
        .then(function(response) {
          that.loading = false
          Notification.success('统计完成')
          that.Point_Layer(response.data)
          console.log(response.data)
        })
        .catch(function(error) {
          that.loading = false
          console.log(error)
        })
    },
    Point_Layer(list) {
      if (list.length === 0) {
        Notification.warning('此时间段查询不到数据')
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
      let PointVectorSource = new VectorSource({})
      let previous = null
      this.map.getLayers().forEach(function(v) {
        if (v.get('layerName') === 'points_layer') {
          previous = v
        }
      })
      if (previous === null) {
        const pointsLayer = new WebGLPointsLayer({
          source: PointVectorSource,
          style: predefinedStyles['circles'],
          disableHitDetection: true
        })
        pointsLayer.set('layerName', 'points_layer')
        this.map.addLayer(pointsLayer)
      } else {
        PointVectorSource = previous.getSource()
        PointVectorSource.clear()
      }
      PointVectorSource.addFeatures(features)
      this.map.getView().fit(PointVectorSource.getExtent(), this.map.getSize())
    },
    drawPoly() {
      let previous = null
      this.map.getLayers().forEach(function(v) {
        if (v.get('layerName') === 'section_layer') {
          previous = v
        }
      })
      let section_source = new VectorSource({ wrapX: false })
      if (previous == null) {
        // 画断面图层
        const vector1 = new VectorLayer({
          source: section_source,
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
        vector1.set('layerName', 'section_layer')
        this.map.addLayer(vector1)
      } else {
        section_source = previous.getSource()
      }
      const draw = new Draw({
        type: 'Polygon',
        source: section_source
        // geometryFunction: createBox()
      })
      draw.on('drawstart', function() {
        section_source.clear()
      })
      const that = this
      // 监听线绘制结束事件，获取坐标
      draw.on('drawend', function(event) {
        // event.feature 就是当前绘制完成的线的Feature
        that.map.removeInteraction(draw)
        const coord = event.feature.getGeometry().getCoordinates()
        // console.log(coord)
        that.textarea = JSON.stringify(coord)
        const line = turf.lineString(coord[0])
        const buffer = turf.buffer(line, 0.5, { units: 'kilometers' })
        const source = new VectorSource({})
        source.addFeature(new Format().readFeature(line))
        source.addFeature(new Format().readFeature(buffer))
        const buffer_layer = new VectorLayer({ source: source, zIndex: 100 })
        that.map.addLayer(buffer_layer)
        that.buffer_area.in = buffer.geometry.coordinates[1]
        that.buffer_area.out = buffer.geometry.coordinates[0]
        console.log(buffer)
        /** buffer结构
         *{
         *geometry{
         *   coordinates{
         *      0{[[],[]]},1{[[],[]]}
         *   }
         *}
         *}
         */
      })
      this.map.addInteraction(draw)
    }
  }
}
</script>

<style scoped>

</style>
