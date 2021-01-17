<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-row-hover="false"
        show-index
        index-text="#"
        border
        class="treeTable"
      >
        <template v-slot:isok="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen;"
          ></i>
          <i
            class="el-icon-error"
            v-else-if="scope.row.cat_deleted === true"
            style="color:red;"
          ></i>
        </template>
        <template v-slot:level="scope">
          <el-tag type="primary" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" v-else-if="scope.row.cat_level === 2"
            >三级</el-tag
          >
        </template>
        <template v-slot:operation="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="showEditCateDialog(scope.row)"
            >编辑</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="removeCateById(scope.row)"
            >删除</el-button
          >
        </template>
      </tree-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClose"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="90px"
        class="demo-ruleForm"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChange"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分类对话框 -->
    <el-dialog
      title="修改分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClose"
    >
      <el-form
        :model="editCateForm"
        :rules="addCateFormRules"
        ref="editCateFormRef"
        label-width="90px"
        class="demo-ruleForm"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitEditCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //   总数据条数
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          prop: 'cat_deleted',
          type: 'template',
          //   表示当前列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          prop: 'cat_level',
          type: 'template',
          template: 'level'
        },
        {
          label: '操作',
          type: 'template',
          template: 'operation'
        }
      ],
      // 添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请填写分类名称', trigger: 'blur' }
        ]
      },
      // 分类列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: true
      },
      // 选中的父级分类的数组
      selectedKeys: [],
      // 控制修改分类对话框的显示与隐藏
      editCateDialogVisible: false,
      editCateForm: {
        cat_id: 0,
        cat_name: ''
      }
    }
  },
  methods: {
    //   获取商品列表
    getCateList: async function() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.$message.success('获取商品列表成功')
      //   console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange: function(newpage) {
      this.queryInfo.pagesize = newpage
      this.getCateList()
    },
    handleCurrentChange: function(newcurrent) {
      this.queryInfo.pagenum = newcurrent
      this.getCateList()
    },
    // 点击显示添加分类对话框
    showAddCateDialog: async function() {
      this.addCateDialogVisible = true
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      // console.log(res.data)
      this.parentCateList = res.data
    },
    // 选择项发生变化触发
    parentCateChange: function() {
      // console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 添加分类
    addCate: function() {
      // console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 添加分类对话框关闭事件
    addCateDialogClose: function() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 点击显示修改分类对话框
    showEditCateDialog: function(cate) {
      console.log(cate.cat_name)
      this.editCateForm.cat_name = cate.cat_name
      this.editCateForm.cat_id = cate.cat_id
      this.editCateDialogVisible = true
    },
    // 修改分类对话框关闭事件
    editCateDialogClose: function() {
      this.$refs.editCateFormRef.resetFields()
    },
    submitEditCate: function() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(
          `categories/${this.editCateForm.cat_id}`,
          {
            cat_name: this.editCateForm.cat_name
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新分类名称失败')
        }
        this.$message.success('更新分类名称成功')
        this.editCateDialogVisible = false
        this.getCateList()
      })
    },
    // 根据id删除分类
    removeCateById: async function(cateInfo) {
      const confirmResult = await this.$confirm(
        `将要删除${cateInfo.cat_name}分类，是否继续`,
        '温馨提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(e => e)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(
        `categories/${cateInfo.cat_id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败')
      }
      this.$message.success('删除分类成功')
      if (this.cateList.length === 1) {
        this.queryInfo.pagenum--
      }
      this.getCateList()
    }
  },
  created() {
    this.getCateList()
  }
}
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 10px;
}
.el-cascader {
  width: 100%;
}
</style>
