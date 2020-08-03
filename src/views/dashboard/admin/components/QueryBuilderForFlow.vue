<template>
  <div id="draggable1" class="box box-1">
    <el-card v-show="isShow" class="box-card">
      <div slot="header" style="text-align: left">
        <span>查询条件框</span>
        <el-button style="float: right; padding: 3px 0" type="text" @click="FunClose"><i class="el-icon-close" /></el-button>
      </div>
      <div class="block" style="margin: auto;">
        <el-divider><el-tag>设置时间</el-tag></el-divider>
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
        />
        <el-button @click="Query_trajectory">轨迹点查询</el-button>
      </div>
      <el-divider><el-tag>设置截面</el-tag></el-divider>
      <div>
        <el-tooltip class="item" effect="dark" content="使用鼠标左键单击地图绘制截面" placement="bottom-start">
          <el-button @click="Draw_lines">鼠标绘制</el-button>
        </el-tooltip>
        <el-button @click="DrawEnd">结束绘制</el-button>
        <el-button @click="QueryFlow">流量统计</el-button>
        <el-collapse>
          <el-collapse-item title="截面端点经纬度" name="1">
            <el-tag type="info">格式: 130.11,30.24</el-tag>
            <el-form :model="demo" :rules="demoRules">
              <el-form-item prop="title">
                <md-input v-model="demo.start" icon="el-icon-search" name="title" placeholder="起点经纬度">
                  起点
                </md-input>
                <md-input v-model="demo.end" icon="el-icon-search" name="title" placeholder="终点经纬度">
                  终点
                </md-input>
              </el-form-item>
            </el-form>
          </el-collapse-item>
        </el-collapse>
        <h3>统计结果</h3>
        <el-table
          :data="tableData"
          border
          height="300"
          style="width: 70%"
        >
          <el-table-column
            fixed
            prop="mmsi"
            label="编号"
            width="70"
          />
          <el-table-column
            prop="location[0]"
            label="经度"
            width="120"
          />
          <el-table-column
            prop="location[1]"
            label="纬度"
            width="120"
          />
          <el-table-column
            prop="time"
            label="时间"
            width="120"
          />
        </el-table>
      </div>
    </el-card>
    <el-button v-show="!isShow" @click="FunClose"><i class="el-icon-set-up" /></el-button>
    <!--    <drag-dialog-demo />-->
  </div>
</template>
<script>
import MdInput from '@/components/MDinput'
import axios from 'axios'
import * as turf from '@turf/turf'
// import DragDialogDemo from '@/views/components-demo/drag-dialog'
export default {
  name: 'QueryBuilderForFlow',
  components: {
    // DragDialogDemo,
    MdInput
  },
  data() {
    const validate = (rule, value, callback) => {
      if (value.length !== 6) {
        callback(new Error('请输入六个字符'))
      } else {
        callback()
      }
    }
    return {
      isShow: true, // 查询框体是否折叠
      demo: { // 截面经纬度信息及约束条件
        title: '',
        start: '',
        end: ''
      },
      demoRules: {
        title: [{ required: true, trigger: 'change', validator: validate }]
      },
      pickerOptions: { // 日期选择器配置
        shortcuts: [{
          text: '2017年2月',
          onClick(picker) {
            const end = new Date('2017-02-07')
            const start = new Date('2017-02-06')
            // start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
            picker.$emit('pick', [start, end])
          }
        // }, {
        //   text: '最近一个月',
        //   onClick(picker) {
        //     const end = new Date()
        //     const start = new Date()
        //     start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
        //     picker.$emit('pick', [start, end])
        //   }
        // }, {
        //   text: '最近三个月',
        //   onClick(picker) {
        //     const end = new Date()
        //     const start = new Date()
        //     start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
        //     picker.$emit('pick', [start, end])
        //   }
        }, {
          text: '2018年10月',
          onClick(picker) {
            const end = new Date('2018-10-03')
            const start = new Date('2018-10-01')
            // start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      },
      query: { // axios传参
        line: this.demo,
        date: this.value2
      },
      value1: '',
      value2: '', // 日期选择器的值
      tableData: []
    }
  },
  computed: { // 计算属性，在截面变化时计算中心点与截面长度
    centerPoint: function() { // 由于组件限制，this.demo.start虽然是Number数组形式，但值都是String类型，使用split拆分再强制类型转换就可以得到一个Number数组形式的中心点
      const StartPoint = this.demo.start.split(',')
      const EndPoint = this.demo.end.split(',')
      const tmp = new Array(2)
      tmp[0] = (Number(StartPoint[0]) + Number(EndPoint[0])) / 2.0.toFixed(4)
      tmp[1] = (Number(StartPoint[1]) + Number(EndPoint[1])) / 2.0.toFixed(4)// 需要格式化
      return tmp
    },
    lineLength: function() { // 计算经纬度间距离不能使用平面坐标系,使用turf插件计算
      if (this.demo.start.equal === '' || this.demo.end === '') {
        return 0
      } else {
        const StartPoint = this.demo.start.split(',')
        const EndPoint = this.demo.end.split(',')
        const options = { units: 'kilometers' } // kilometers,miles
        const distance = turf.distance(StartPoint, EndPoint, options).toFixed(4) // 注意格式化，需要限制小数点的位数
        return distance
      }
    }
  },
  methods: {
    FunClose: function() { // 控制查询框显示
      this.isShow = !this.isShow
    },
    Draw_lines: function() {
      // 调用父组件map里面的draw_line方法绘制截面
      this.$parent.draw_line()
      this.$parent.open_msg('warning', '绘制完成请点击结束绘制按钮')
    },
    DrawEnd() { // 绘制完成事件
      this.$parent.draw_end()
      this.demo.start = '' + this.$parent.StartPoint // 此处限制为String类型
      this.demo.end = '' + this.$parent.EndPoint
      console.log('LineLength: ' + this.lineLength + 'km')
      this.$parent.open_msg('success', 'LineLength: ' + this.lineLength + 'km')
    },
    QueryFlow: function() { // 截面流量统计方法，异步请求后台
      // 1.axois get方法无参 在异步请求作用域内this已经发生改变了，不是指向原vue,所以使用that保存当初状态
      // const that = this
      // axios.get('http://localhost:5003/table').then(function(response) {
      //   that.value1 = response.data
      //   console.log(that.value1)
      // })
      // 2.axios post方法传json参数：时间段、截面中心点、截面长度
      if (this.value2 === '' || this.demo.end === '' || this.demo.start === '') {
        this.$parent.open_msg('warning', '请设置时间与截面')
        return
      }
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1],
        start_point: this.demo.start,
        end_point: this.demo.end,
        center_point: this.centerPoint,
        line_length: this.lineLength / 2.0// 这不是半径，修改！！
      }
      const that = this
      axios.post('http://localhost:5003/queryByTimeAndLocation', params)
        .then(function(response) {
          that.tableData = response.data
          that.$parent.open_msg('success', '流量:' + response.data.length)
          console.log('流量' + response.data.length)
        })
        .catch(function(error) {
          console.log(error)
        })
      // 3.axios get路径传参
      // axios.get('http://localhost:5003/queryByTime1?timein=' + this.value2[0] + '&timeout=' + this.value2[1] + '&start_point=' + this.demo.start + '&end_point=' + this.demo.end).then(function(response) {
      //   console.log(response.data)
      // })
    },
    Query_trajectory: function() {
      if (this.value2 === '') {
        this.$parent.open_msg('warning', '请选择时间段')
        return
      }
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1]
      }
      const that = this
      axios.post('http://localhost:5003/queryByTime', params)
        .then(function(response) {
          that.$parent.point_layer(response.data.content)
          that.$parent.open_msg('success', '查询成功')
        })
        .catch(function(error) {
          console.log(error)
        })
    }
  }

}
</script>

<style scoped>
  .item {
    margin-bottom: 18px;
  }
  .box-card {
    width: 480px;
  }
  .box {
    position: absolute;
    background: #ffffff;
    border-radius: 10px;
    /*z-index: 10;*/
    left: 10px;
    /*right: 0;*/
    top: 7%;
    float-displace: auto;
    margin:0 auto;
    opacity: 0.9;
    /*margin-left: auto;*/
    /*margin-right: auto;*/
    z-index: 999;
    text-align: center;
  }
  .box-1{
    width:  auto;
    height: auto;
    display: block;
  }
  a:hover{cursor: pointer}
</style>
