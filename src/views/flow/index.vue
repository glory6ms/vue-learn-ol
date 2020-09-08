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
    <div v-show="result_show" id="my_table">
      <el-collapse v-model="activeName" accordion>
        <el-collapse-item title="统计结果" name="1">
          <el-row>
            <el-col :span="6">
              <el-table
                :data="tableData1"
                style="width: 100%"
              >
                <el-table-column
                  prop="kind"
                  label="船舶种类"
                  width="180"
                />
                <el-table-column
                  prop="num"
                  label="数量"
                  width="180"
                />
              </el-table>
            </el-col>
            <el-col :span="6">
              <el-table
                :data="tableData3"
                style="width: 100%"
              >
                <el-table-column
                  prop="state"
                  label="航向状态"
                  width="180"
                />
                <el-table-column
                  prop="num"
                  label="数量"
                  width="180"
                />
              </el-table>
            </el-col>
            <el-col :span="6">
              <el-table
                :data="tableData4"
                style="width: 100%"
              >
                <el-table-column
                  prop="aisType"
                  label="ais类型"
                  width="180"
                />
                <el-table-column
                  prop="num"
                  label="数量"
                  width="180"
                />
              </el-table>
            </el-col>
            <el-col :span="6">
              <el-table
                :data="tableData2"
                style="width: 100%"
              >
                <el-table-column
                  prop="speed"
                  label="船舶航速"
                  width="180"
                />
                <el-table-column
                  prop="num"
                  label="数量"
                  width="180"
                />
              </el-table>
            </el-col>
          </el-row>
        </el-collapse-item>
        <el-collapse-item title="动态信息" name="4">
          <el-table
            v-loading="loading"
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
        </el-collapse-item>
      </el-collapse>
    </div>

  </div>
</template>

<script>
import * as turf from '@turf/turf'
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
      tableData: [],
      activeName: '4',
      tableData1: [{
        kind: '货船',
        num: 85
      }, {
        kind: '拖轮',
        num: 24
      }, {
        kind: '油轮',
        num: 37
      }, {
        kind: '执法船',
        num: 5
      }],
      tableData2: [{
        speed: '0-4',
        num: 15
      }, {
        speed: '4-8',
        num: 24
      }, {
        speed: '8-12',
        num: 37
      }, {
        speed: '>12',
        num: 5
      }],
      tableData3: [{
        state: '机动航行',
        num: 124
      }, {
        state: '停泊',
        num: 58
      }],
      tableData4: [{
        aisType: 'class A',
        num: 168
      }, {
        aisType: 'class B',
        num: 134
      }]
    }
  },
  computed: {
    centerPoint() {
      const startPoint = this.line.start.split(',')
      const endPoint = this.line.end.split(',')
      const midPoint = turf.midpoint(turf.point(startPoint), turf.point(endPoint))
      return midPoint.geometry.coordinates
    },
    lineLength() {
      if (this.line.start === '' || this.line.end === '') {
        return 0
      } else {
        const startPoint = this.line.start.split(',')
        const endPoint = this.line.end.split(',')
        const options = { units: 'kilometers' } // kilometers,miles
        const distance = turf.distance(startPoint, endPoint, options).toFixed(5) // 注意格式化，需要限制小数点的位数
        return distance / 2
      }
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
          Notification.error('查询失败')
          that.loading = false
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
      console.log(this.lineLength)
      if (this.value2 === '' || this.line.end === '' || this.line.start === '' || this.line.end === ',' || this.line.start === ',') {
        Notification.warning('请设置时间与截面')
        return
      }
      this.loading = true
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1],
        start_point: this.line.start,
        end_point: this.line.end,
        center_point: this.centerPoint,
        line_length: this.lineLength
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
</style>
