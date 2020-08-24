<template>
  <div>
    <SearchDialog>
      <template v-slot:header>
        <span>截面流量统计</span>
      </template>
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
        <div v-loading="loading">
          <el-collapse>
            <el-collapse-item title="自定义截面端点经纬度" name="1">
              <el-tag type="info">格式: 130.11,30.24</el-tag>
              <el-input
                v-model="line.start"
                placeholder="起点"
                clearable
              />
              <el-input
                v-model="line.end"
                placeholder="终点"
                clearable
              />
            </el-collapse-item>
          </el-collapse>
          <div id="btn_group">
            <el-button title="单击地图滑动鼠标再次单击结束绘制" @click="Draw_Lines">鼠标绘制</el-button>
            <el-button title="查询截面流量" @click="Query_Flow">流量统计</el-button>
          </div>
        </div>
      </template>
    </SearchDialog>
    <el-table
      v-show="result_show"
      id="my_table"
      :data="tableData"
      border
      height="200"
    >
      <el-table-column
        fixed
        prop="mmsi"
        label="编号"
      />
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
  </div>
</template>

<script>
import SearchDialog from '@/components/SearchDialog/index'
import axios from 'axios'
import { Notification } from 'element-ui'
import Feature from 'ol/Feature'
import Point from 'ol/geom/Point'
import VectorSource from 'ol/source/Vector'
import WebGLPointsLayer from 'ol/layer/WebGLPoints'
import VectorLayer from 'ol/layer/Vector'
import { Circle, Fill, Stroke, Style } from 'ol/style'
import Draw from 'ol/interaction/Draw'
export default {
  name: 'TrackFlow',
  components: { SearchDialog },
  props: ['map'],
  data() {
    return {
      result_show: false,
      loading: false,
      line: { // 截面经纬度信息及约束条件
        start: '',
        end: ''
      },
      pickerOptions: { // 日期选择器配置
        shortcuts: [{
          text: '2018年10月',
          onClick(picker) {
            const end = new Date('2018-10-03')
            const start = new Date('2018-10-01')
            picker.$emit('pick', [start, end])
          }
        }]
      },
      value2: '', // 日期选择器的值
      tableData: []
    }
  },
  created() {
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
      const that = this
      this.loading = true
      axios.post('http://10.138.100.254:5003/queryByTime', params)
        .then(function(response) {
          if (response.data.length === 0) {
            Notification.warning('查询结果数为零')
            return
          }
          that.Point_Layer(response.data)
          that.loading = false
          Notification.success('查询成功!')
        })
        .catch(function(error) {
          Notification.error('查询失败，请重新输入时间段')
          console.log(error)
        })
    },
    Draw_Lines: function() {
      // 判断是否为第一次添加截面图层,必须先删除clear layer的source再remove
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
      const draw1 = new Draw({
        source: section_source,
        type: 'LineString',
        maxPoints: 2
      })
      draw1.on('drawstart', function() {
        section_source.clear()
      })
      const that = this
      draw1.on('drawend', function(event) {
        // 截面线段绘制完成
        that.map.removeInteraction(draw1)
        that.line.start = '' + event.feature.getGeometry().getCoordinates()[0]
        that.line.end = '' + event.feature.getGeometry().getCoordinates()[1]
      })
      this.map.addInteraction(draw1)
    },
    Query_Flow: function() {
      if (this.value2 === '' || this.line.end === '' || this.line.start === '' || this.line.end === ',' || this.line.start === ',') {
        Notification.warning('请设置时间与截面')
        return
      }
      this.loading = true
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1],
        start_point: this.line.start,
        end_point: this.line.end
      }
      const that = this
      axios.post('http://10.138.100.254:5003/queryByTimeAndLocation', params)
        .then(function(response) {
          that.tableData = response.data
          Notification.success('流量:' + response.data.length)
          that.result_show = true
          that.loading = false
        })
        .catch(function(error) {
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
    map_click() {
      let end_point = new Array(2)
      let start_point = new Array(2)
      let count = 1
      let click_flag = 1
      const that = this
      this.map.on('singleclick', function(evt) {
        if (click_flag === 1) {
          if (count % 2 === 0) {
            // eslint-disable-next-line no-unused-vars
            end_point = [evt.coordinate[0].toFixed(4), evt.coordinate[1].toFixed(4)]
            console.log('end:' + end_point)
            that.line.end = '' + end_point
            click_flag = 0
            Notification.success('请点击流量统计按钮')
          } else {
            // eslint-disable-next-line no-unused-vars
            start_point = [evt.coordinate[0].toFixed(4), evt.coordinate[1].toFixed(4)]
            console.log('start:' + start_point)
            that.line.start = '' + start_point
          }
          count = count + 1
        }
      })
    }
  }
}
</script>

<style scoped>
#my_table{
  position: absolute;
  /*z-index: 10;*/
  bottom: 2px;
  float-displace: auto;
  opacity: 0.9;
  z-index: 999;
  text-align: center;
}
</style>
