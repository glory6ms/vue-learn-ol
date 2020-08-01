<template>
  <div id="draggable1" class="box box-1">
    <el-card v-show="isShow" class="box-card">
      <div slot="header" style="text-align: left">
        <span>查询条件框</span>
        <el-button style="float: right; padding: 3px 0" type="text" @click="FunClose"><i class="el-icon-close" /></el-button>
      </div>
      <div class="block" style="margin: auto;">
        <!--        <el-tag>时间段选择</el-tag>-->
        <el-divider><el-tag>设置时间</el-tag></el-divider>
        <div class="demonstration">值：{{ value2 }}</div>
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
            <el-tag type="info">格式: [130.11,30.24]</el-tag>
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
      </div>
    </el-card>
    <el-button v-show="!isShow" @click="FunClose"><i class="el-icon-set-up" /></el-button>
  </div>
</template>
<script>
import MdInput from '@/components/MDinput'
import axios from 'axios'
export default {
  name: 'QueryBuilderForFlow',
  components: {
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
      demo: {
        title: '',
        start: '',
        end: ''
      },
      demoRules: {
        title: [{ required: true, trigger: 'change', validator: validate }]
      },
      pickerOptions: { // 日期选择器配置
        shortcuts: [{
          text: '最近一周',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '最近一个月',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '最近三个月',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      },
      query: {
        line: this.demo,
        date: this.value2
      },
      value1: '',
      value2: ''
    }
  },
  methods: {
    FunClose: function() { // 控制查询框显示
      this.isShow = !this.isShow
    },
    Draw_lines: function() {
      // 调用父组件map里面的draw_line方法绘制截面
      this.$parent.draw_line()
    },
    DrawEnd() { // 绘制完成事件
      this.$parent.draw_end()
      this.demo.start = '[' + this.$parent.StartPoint + ']'
      this.demo.end = '[' + this.$parent.EndPoint + ']'
    },
    QueryFlow: function() { // 截面流量统计方法，异步请求后台
      // 1.axois get方法无参 在异步请求作用域内this已经发生改变了，不是指向原vue,所以使用that保存当初状态
      // const that = this
      // axios.get('http://localhost:5003/table').then(function(response) {
      //   that.value1 = response.data
      //   console.log(that.value1)
      // })
      // 2.axios post方法传json
      const params = {
        timein: this.value2[0],
        timeout: this.value2[1],
        start_point: this.demo.start,
        end_point: this.demo.end
      }
      axios.post('http://localhost:5003/queryByTime', params)
        .then(function(response) {
          console.log(response.data)
        })
        .catch(function(error) {
          console.log(error)
        })
      // 3.axios get路径传参
      // axios.get('http://localhost:5003/queryByTime1?timein=' + this.value2[0] + '&timeout=' + this.value2[1] + '&start_point=' + this.demo.start + '&end_point=' + this.demo.end).then(function(response) {
      //   console.log(response.data)
      // })
    }
  }

}
</script>

<style scoped>
  .text {
    font-size: 14px;
  }

  .item {
    margin-bottom: 18px;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
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
