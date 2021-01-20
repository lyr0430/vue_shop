<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="18">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            class="input-with-select"
            @clear="searchGoods"
            @change="searchGoods"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="searchGoods"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>
      <el-table :data="goodsTableData" style="width: 100%" stripe border>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="goods_name" label="商品名称" width="500">
        </el-table-column>
        <el-table-column prop="goods_price" label="商品价格"> </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量">
        </el-table-column>
        <el-table-column
          prop="add_time"
          label="创建时间"
          :formatter="timeFormat"
        >
          <!-- <template v-slot="scope">
            {{scope.row.add_time | format('yyyy-MM-dd hh:mm:ss')}}
        </template> -->
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template>
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加商品对话框 -->
  </div>
</template>
<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      //   商品总数
      total: 0,
      goodsTableData: []
    }
  },
  methods: {
    getGoodsList: async function() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.$message.success('获取商品列表成功')
      //   console.log(res)
      this.goodsTableData = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange: function(newPageSize) {
      this.queryInfo.pagesize = newPageSize
      this.getGoodsList()
    },
    handleCurrentChange: function(newPageNum) {
      this.queryInfo.pagenum = newPageNum
      this.getGoodsList()
    },
    // 日期格式化
    timeFormat: function(row) {
      function dateFormat(date, format) {
        if (typeof date === 'string') {
          const mts = date.match(/(\/Date\((\d+)\)\/)/)
          if (mts && mts.length >= 3) {
            date = parseInt(mts[2])
          }
        }
        date = new Date(date)
        if (!date || date.toUTCString() === 'Invalid Date') {
          return ''
        }
        const map = {
          M: date.getMonth() + 1, // 月份
          d: date.getDate(), // 日
          h: date.getHours(), // 小时
          m: date.getMinutes(), // 分
          s: date.getSeconds(), // 秒
          q: Math.floor((date.getMonth() + 3) / 3), // 季度
          S: date.getMilliseconds() // 毫秒
        }
        format = format.replace(/([yMdhmsqS])+/g, function(all, t) {
          let v = map[t]
          if (v !== undefined) {
            if (all.length > 1) {
              v = '0' + v
              v = v.substr(v.length - 2)
            }
            return v
          } else if (t === 'y') {
            return (date.getFullYear() + '').substr(4 - all.length)
          }
          return all
        })
        return format
      }
      return dateFormat(row.add_time, 'yyyy-MM-dd hh:mm:ss')
    },
    // 搜索商品
    searchGoods: function() {
      this.queryInfo.pagenum = 1
      this.getGoodsList()
    },
    goAddPage: function() {
      this.$router.push('/goods/add')
    }
  },
  created() {
    this.getGoodsList()
  },
  watch: {
    'queryInfo.query': function(val) {
      if (val.trim().length === 0) {
        this.queryInfo.pagenum = 1
        this.getGoodsList()
      }
    }
  }
}
</script>
<style lang="less" scoped></style>
