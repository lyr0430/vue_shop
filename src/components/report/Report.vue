<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <div id="main" style="width: 100%;height:450px;"></div>
    </el-card>
  </div>
</template>
<script>
import * as echarts from 'echarts'
// let echarts = require('echarts')
import _ from 'lodash'
export default {
  data() {
    return {
      //   echartsOptions: {}
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  methods: {
    // getReport: async function() {
    //   console.log(this.echartsOptions)
    // },
    initEcharts: async function() {
      const { data: res } = await this.$http.get('reports/type/1')
      //   console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取折线图数据失败')
      }
      //   this.echartsOptions = res.data
      var myChart = echarts.init(document.getElementById('main'))
      const result = _.merge(res.data, this.options)
      myChart.setOption(result)
      //   console.log(123)
    }
  },
  //   此时页面上的dom元素已经被渲染完毕
  mounted() {
    this.initEcharts()
  }
}
</script>
<style lang="less" scoped></style>
