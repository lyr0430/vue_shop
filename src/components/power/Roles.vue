<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRoleDialog"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <el-table :data="rolesList" style="width: 100%" border stripe>
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :class="['bd_bottom', i1 === 0 ? 'bd_top' : '', 'vcenter']"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bd_top', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="roleName" label="角色"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditRoleDialog(scope.row)"
              >编辑</el-button
            >
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >
            <!-- 分配权限按钮 -->
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="50%"
      @close="addRoleDialogClosed"
    >
      <el-form
        :model="addRoleForm"
        :rules="addRoleRules"
        ref="addRoleFormRef"
        label-width="90px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色对话框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="editRoleDialogVisible"
      width="50%"
      @close="editRoleDialogClosed"
    >
      <el-form
        :model="editRoleForm"
        :rules="addRoleRules"
        ref="editRoleFormRef"
        label-width="90px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      //   所有权限的数据
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //   默认选中的节点id值数组
      defKeys: [],
      //   当前即将分配权限的角色id
      roleId: '',
      // 添加角色对话框的显示与隐藏
      addRoleDialogVisible: false,
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      addRoleRules: {
        roleName: [
          { required: true, message: '请填写角色名称', trigger: 'blur' }
        ]
      },
      // 编辑角色对话框的显示与隐藏
      editRoleDialogVisible: false,
      // 编辑角色信息
      editRoleForm: {},
      editRoleId: ''
    }
  },
  methods: {
    getRolesList: async function() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.$message.success('获取角色列表成功')
      this.rolesList = res.data
      //   console.log(res.data)
    },
    // 根据id删除权限
    removeRightById: async function(role, rightId) {
      //   console.log(roleId, rightId)
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限，是否继续',
        '温馨提醒',
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
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      //   this.getRolesList()
      role.children = res.data
    },
    // 展示修改分配权限的对话框
    showSetRightDialog: async function(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      this.rightsList = res.data
      //   console.log(this.rightsList)
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys中
    getLeafKeys: function(node, arr) {
      // 如果当前node节点不包含children，则为三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        return this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed: function() {
      this.defKeys = []
    },
    allotRights: async function() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedNodes()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        {
          rids: idStr
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 显示添加角色对话框
    showAddRoleDialog: function() {
      this.addRoleDialogVisible = true
    },
    // 添加角色
    addRole: function() {
      this.$refs.addRoleFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) {
          return this.$message.error('请检查验证未通过的数据项')
        }
        // console.log(this.addRoleForm)
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.addRoleDialogVisible = false
        this.$message.success('添加角色成功')
        this.getRolesList()
      })
    },
    // 添加角色对话框关闭
    addRoleDialogClosed: function() {
      this.addRoleForm = {}
    },
    // 根据id删除角色
    removeRoleById: async function(id) {
      const confirmResult = await this.$confirm(
        '确认要删除该角色么，是否继续',
        '温馨提醒',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(e => e)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败')
      }
      this.getRolesList()
      this.$message.success('删除角色成功')
    },
    // 点击按钮显示编辑角色对话框
    showEditRoleDialog: function(role) {
      // console.log(x)
      this.editRoleId = role.id
      this.editRoleForm = role
      this.editRoleDialogVisible = true
    },
    // 提交角色编辑
    editRole: async function() {
      const { data: res } = await this.$http.put(
        `roles/${this.editRoleId}`,
        this.editRoleForm
      )
      if (res.meta.status !== 200) {
        return this.$message.error('编辑角色信息失败')
      }
      this.$message.success('编辑角色信息成功')
      this.editRoleDialogVisible = false
      this.getRolesList()
    },
    // 编辑角色对话框关闭
    editRoleDialogClosed: function() {
      this.editRoleForm = {}
    }
  },
  created() {
    this.getRolesList()
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bd_top {
  border-top: 1px solid #eee;
}
.bd_bottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
