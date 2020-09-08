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
    <div v-show="result_show" id="resultPanel">
      <el-card class="box-card">
        <div slot="header" style="text-align: left">
          <span>活跃度分析结果</span>
          <el-button style="float: right; padding: 3px 0" type="text" @click="result_show=!result_show"><i class="el-icon-close" /></el-button>
        </div>
        <div><i class="el-icon-ship" />: {{ num }} 艘</div>
        <!--      <el-divider><el-tag type="info" effect="plain">设置截面</el-tag></el-divider>-->
        <div id="myPie" style="height: 250px; width: 250px" />
        <div id="myLine" style="height: 250px; width: 250px" />
        <el-table
          v-loading="loading"
          :data="tableData"
          border
          height="200"
        >
          <el-table-column
            prop="location[0]"
            label="经度"
          />
          <el-table-column
            prop="location[1]"
            label="纬度"
          />
          <el-table-column
            prop="time"
            label="时间"
          />
        </el-table>
      </el-card>
    </div>
    <!--    <div v-show="result_show" id="my_table">-->
    <!--      -->
    <!--    </div>-->
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
import echarts from 'echarts'
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
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2017年7月1',
          onClick(picker) {
            const end = new Date('2017-07-10')
            const start = new Date('2017-07-01')
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2017年7月10',
          onClick(picker) {
            const end = new Date('2017-07-20')
            const start = new Date('2017-07-10')
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2017年7月20',
          onClick(picker) {
            const end = new Date('2017-07-31')
            const start = new Date('2017-07-20')
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2018年1月1',
          onClick(picker) {
            const end = new Date('2018-01-20')
            const start = new Date('2018-01-01')
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '2018年1月20',
          onClick(picker) {
            const end = new Date('2018-01-30')
            const start = new Date('2018-01-20')
            picker.$emit('pick', [start, end])
          }
        }]
      },
      value2: '', // 日期选择器的值
      buffer_area: { // 截面经纬度信息及约束条件
        in: [],
        out: []
      },
      loading: false,
      tableData: []
    }
  },
  computed: {
    num() {
      return this.tableData.length
    }
  },
  mounted() {
    this.init_pie()
    this.init_line()
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
        out: this.buffer_area.out,
        line: JSON.parse(this.textarea)
      }
      this.loading = true
      const that = this
      axios.post('http://localhost:5003/QueryByPolygonRange', params)
        .then(function(response) {
          that.loading = false
          Notification.success('统计完成')
          // console.log(response.data)
          that.Point_Layer(response.data.pois)
          that.tableData = response.data.result
          that.result_show = true
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
      let previousBuffer = null
      this.map.getLayers().forEach(function(v) {
        if (v.get('layerName') === 'section_layer') {
          previous = v
        }
        if (v.get('layerName') === 'buffer_layer') {
          previousBuffer = v
        }
      })
      let section_source = new VectorSource({ wrapX: false })// polygon feature的source
      let source = new VectorSource({})
      if (previous == null || previousBuffer == null) {
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
        const buffer_layer = new VectorLayer({ source: source, zIndex: 100 })
        buffer_layer.set('layerName', 'buffer_layer')
        this.map.addLayer(buffer_layer)
      } else {
        section_source = previous.getSource()
        source = previousBuffer.getSource()
      }
      const draw = new Draw({
        type: 'Polygon',
        source: section_source
        // geometryFunction: createBox()
      })
      draw.on('drawstart', function() {
        section_source.clear()
        source.clear()
      })
      const that = this
      // 监听线绘制结束事件，获取坐标
      draw.on('drawend', function(event) {
        // event.feature 就是当前绘制完成的线的Feature
        that.map.removeInteraction(draw)
        const coord = event.feature.getGeometry().getCoordinates()
        // that.textarea = JSON.stringify(coord)
        that.textarea = JSON.stringify(coord[0])
        const line = turf.lineString(coord[0])
        const buffer = turf.buffer(line, 0.5, { units: 'kilometers' })
        source.addFeature(new Format().readFeature(line))
        source.addFeature(new Format().readFeature(buffer))
        that.buffer_area.in = buffer.geometry.coordinates[1]
        that.buffer_area.out = buffer.geometry.coordinates[0]
        // console.log(buffer)
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
    },
    init_pie() {
      const myChart = echarts.init(document.getElementById('myPie'))
      myChart.setOption({
        title: {
          subtext: '船舶种类占比',
          left: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{b} : {c} ({d}%)'
        },
        series: [
          {
            type: 'pie',
            radius: '25%',
            center: ['50%', '50%'],
            data: [
              { value: 320, name: '货船' },
              { value: 240, name: '拖轮' },
              { value: 149, name: '油轮' },
              { value: 100, name: '执法船' }
            ],
            animationEasing: 'cubicInOut',
            animationDuration: 2600
          }
        ]
      })
    },
    init_line() {
      const myChart = echarts.init(document.getElementById('myLine'))
      myChart.setOption({
        title: {
          subtext: '船舶种类占比',
          left: 'center'
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: ['0-3', '4-6', '周三', '周四', '周五', '周六', '周日']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            type: 'line',
            data: [20, 32, 11, 34, 90, 30, 20]
          }
        ]
      })
    }
  }
}
</script>

<style scoped>
#my_table{
  position: absolute;
  bottom: 2px;
  float-displace: auto;
  opacity: 0.9;
  z-index: 999;
  text-align: center;
  width: 100%;
}
#resultPanel{
  position: absolute;
  right: 2px;
  float-displace: auto;
  opacity: 0.9;
  z-index: 999;
  width: 250px;
}
</style>
