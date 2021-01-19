<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="注意：只允许为第三级分类设置相关参数"
        type="warning"
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 选择商品分类表单 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            v-model="selectedKey"
            :options="cateList"
            :props="cateProps"
            @change="cateChange"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isDisabled"
            @click="showAddDialog"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe style="width: 100%">
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  closable
                  @close="removeVals(index, scope.row)"
                  >{{ item }}</el-tag
                >
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditAttrDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeAttrById(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态参数面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isDisabled"
            @click="showAddDialog"
            >添加属性</el-button
          >
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe style="width: 100%">
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  closable
                  @close="removeVals(index, scope.row)"
                  >{{ item }}</el-tag
                >
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditAttrDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeAttrById(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose"
    >
      <el-form
        :model="addAttrForm"
        :rules="addAttrFormRules"
        ref="addAttrFormRef"
        label-width="90px"
        class="demo-ruleForm"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addAttrForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addAttr">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose"
    >
      <el-form
        :model="editAttrForm"
        :rules="addAttrFormRules"
        ref="editAttrFormRef"
        label-width="90px"
        class="demo-ruleForm"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editAttrForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editAttr">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      selectedKey: [],
      activeName: 'many',
      //   动态参数数据
      manyTableData: [],
      //   静态属性数据
      onlyTableData: [],
      addDialogVisible: false,
      addAttrForm: {
        attr_name: ''
      },
      addAttrFormRules: {
        attr_name: [
          { required: true, message: '该项是必填项', trigger: 'blur' }
        ]
      },
      editAttrForm: {
        attr_name: ''
      },
      editDialogVisible: false,
      editAttrId: 0
    }
  },
  methods: {
    getCateList: async function() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data
    },
    cateChange: async function() {
      this.getParamsData()
    },
    // tab点击事件
    handleTabClick: function() {
      this.getParamsData()
    },
    getParamsData: async function() {
      if (this.selectedKey.length !== 3) {
        this.selectedKey = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: {
            sel: this.activeName
          }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        //   控制文本框的显示与隐藏
        item.inputVisible = false
        //   文本框中输入的值
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 点击显示对话框
    showAddDialog: function() {
      this.addDialogVisible = true
    },
    // 添加参数
    addAttr: function() {
      this.$refs.addAttrFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addAttrForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    addDialogClose: function() {
      this.$refs.addAttrFormRef.resetFields()
    },
    // 点击显示修改参数对话框
    showEditAttrDialog: function(info) {
      //
      //   console.log(info.attr_id)
      this.editAttrId = info.attr_id
      this.editAttrForm.attr_name = info.attr_name
      this.editDialogVisible = true
    },
    editDialogClose: function() {
      //   this.$refs.editAttrFormRef.resetFields()
      this.editAttrForm.attr_name = ''
    },
    editAttr: function() {
      this.$refs.editAttrFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editAttrId}`,
          {
            attr_name: this.editAttrForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('编辑参数失败')
        }
        this.$message.success('编辑参数成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 根据id删除参数
    removeAttrById: async function(id) {
      const confirmResult = await this.$confirm(
        '正在删除该参数，是否继续？',
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
        `categories/${this.cateId}/attributes/${id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点或按下enter触发
    handleInputConfirm: async function(info) {
      if (info.inputValue.trim().length === 0) {
        info.inputValue = ''
        info.inputVisible = false
        return
      }
      // 用户输入了合法的内容
      info.attr_vals.push(info.inputValue.trim())
      info.inputValue = ''
      info.inputVisible = false
      this.saveAttrVals(info)
    },
    saveAttrVals: async function(row) {
      // 将数据提交到数据库
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败')
      }
      this.$message.success('修改参数项成功')
    },
    showInput: function(info) {
      console.log(info)
      info.inputVisible = true
      //   console.log(info)
      this.$nextTick(() => {
        this.$refs.saveTagInput.focus()
      })
    },
    removeVals: async function(i, row) {
      const confirmResult = await this.$confirm(
        '正在删除该参数，是否继续？',
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
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  created() {
    this.getCateList()
  },
  computed: {
    cateId: function() {
      if (this.selectedKey.length === 3) {
        return this.selectedKey[2]
      }
      return null
    },
    isDisabled: function() {
      if (this.selectedKey.length !== 3) {
        return true
      }
      return false
    },
    // 动态计算标题文本
    titleText: function() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin-right: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
