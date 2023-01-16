<template>
  <div>
    <!-- 面包屑导航 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>权限管理</el-breadcrumb-item>
    <el-breadcrumb-item>角色列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片区 -->
  <el-card class="box-card">
    <!-- 添加角色按钮 -->
    <el-row>
      <el-col>
        <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
      </el-col>
    </el-row>
    <!-- 角色列表区 -->
    <el-table :data="rolesList" stripe border>
      <!-- 展开列 -->
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row :class="['vcenter', bdbottom, i1 === 0 ? 'bdtop':'']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
            <!-- 一级 -->
            <el-col :span="5">
              <el-tag closable  @close="removeRightsById(scope.row,item1.id)">{{item1.authName}}</el-tag>
              <i class="el-icon-caret-right"></i>
            </el-col>
            <!-- 二级 & 三级 -->
            <el-col :span="19">
              <!-- 二级 -->
              <el-row :class="['vcenter', i2 === 0 ? '':'bdtop']" v-for="(item2, i2) in item1.children" :key="item2.id">
                <el-col :span="6">
                  <el-tag type="success" closable  @close="removeRightsById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 三级 -->
                <el-col :span="18">
                  <el-tag closable  @close="removeRightsById(scope.row,item3.id)" type="warning" v-for=" item3 in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作" width="300px" >
        <!-- 权限等级插槽 -->
        <template slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit" @click="editDialogShow(scope.row.id)">编辑</el-button>
          <el-button size="mini" type="warning" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
          <el-button size="mini" type="danger" icon="el-icon-setting" @click="editRightsDialogShow(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
  <!-- 添加角色对话框 -->
  <el-dialog
    title="添加角色"
    :visible.sync="addDialogVisible"
    width="50%"
    @close="addDialogClosed">
    <el-form
      :model="addForm"
      :rules="addFormRules"
      ref="addFormRef"
      label-width="80px">
      <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="addForm.roleName"></el-input>
      </el-form-item>
      <el-form-item label="角色描述" prop="roleDesc">
        <el-input v-model="addForm.roleDesc"></el-input>
      </el-form-item>
      </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="addDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addRole">确 定</el-button>
    </span>
  </el-dialog>
     <!-- 修改角色 对话框 -->
  <el-dialog
    title="修改角色"
    :visible.sync="editDialogVisible"
    width="50%"
    @close="editDialogClosed">
    <el-form
      :model="editForm"
      :rules="editFormRules"
      ref="editFormRef"
      label-width="80px">
      <el-form-item label="角色id" prop="roleId">
        <el-input v-model="editForm.roleId" disabled></el-input>
      </el-form-item>
      <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="editForm.roleName"></el-input>
      </el-form-item>
      <el-form-item label="角色描述" prop="roleDesc">
        <el-input v-model="editForm.roleDesc"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="editDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="editRole">确 定</el-button>
    </span>
  </el-dialog>
  <!-- 分配权限 对话框 -->
  <el-dialog
      title="分配权限"
      :visible.sync="editRightsDialogVisible"
      width="50%"
      @closed="editRightsDialogClosed">
    <el-tree ref="rightsTree" default-expand-all :default-checked-keys="rightsKeyList" show-checkbox :data="rightsList" :props="rightsTreeProps" node-key="id"></el-tree>
    <span slot="footer" class="dialog-footer">
      <el-button @click="editRightsDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="allotRights">确 定</el-button>
    </span>
  </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolesList: [],
      addDialogVisible: false,
      editDialogVisible: false,
      editRightsDialogVisible: false,
      rightsList: [],
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      editForm: {
        roleId: '',
        roleName: '',
        roleDesc: ''
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: false, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: false, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      rightsTreeProps: {
        children: 'children',
        label: 'authName'
      },
      rightsKeyList: [],
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 删除用户
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消了删除，则返回值为字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败')
      }
      this.getRolesList()
      this.$message.success('删除角色成功')
    },
    // 获取角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      this.$message.success('获取角色列表成功')
    },
    // 监听关闭添加用户弹窗
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 监听关闭添加用户弹窗
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 监听关闭角色分配权限弹窗
    editRightsDialogClosed() {
      this.rightsKeyList = []
    },
    // 展示 编辑对话框弹出
    async editDialogShow (id) {
      this.editDialogVisible = true
      // 请求数据
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('数据请求失败')
      }
      this.editForm = res.data
      console.log(res)
      return this.$message.success('数据获取成功')
    },
    // 展示 角色分配权限对话框弹窗
    async editRightsDialogShow (role) {
      // 请求数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('数据请求失败')
      }
      this.rightsList = res.data
      console.log(res.data)
      console.log(role)
      // 要先进行 角色权限列表设置
      this.getRightsKeyList(role, this.rightsKeyList)
      console.log(this.rightsKeyList)
      // 再展示对话框
      this.editRightsDialogVisible = true
      // 获取 role.id 为了后面 进行分配权限 时要用到
      this.roleId = role.id
      return this.$message.success('数据获取成功')
    },
    // 递归设置角色权限列表
    getRightsKeyList(node, arr) {
      // 如果当前 Node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item =>
        this.getRightsKeyList(item, arr))
    },
    // 编辑角色
    editRole () {
      this.$refs.editFormRef.validate(async (valid) => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId,
          { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })
        console.log(res.data)
        if (res.meta.status !== 200) {
          return this.$message.error('更新失败')
        }
        // 关闭表单
        this.editDialogVisible = false
        // 更新用户列表
        this.getRolesList()
        this.$message.success('更新成功')
      })
    },
    // 添加角色
    addRole() {
      this.$refs.addFormRef.validate(async (valid) => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        // 关闭表单
        this.addDialogVisible = false
        // 更新权限列表
        this.getRolesList()
        this.$message.success('添加成功')
      })
    },
    // 删除角色的权限
    async removeRightsById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消了删除，则返回值为字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除用户权限')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户权限失败')
      }
      console.log(res)
      this.$message.success('删除用户权限成功')
      // 不建议刷新界面 因为刷新界面 展开区会收回去
      role.children = res.data
    },
    // 设置角色权限
    async allotRights () {
      // 获取 权限 id
      const keyList = [...this.$refs.rightsTree.getCheckedKeys(), ...this.$refs.rightsTree.getHalfCheckedKeys()]
      const rid = keyList.join(',')
      console.log(rid)
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: rid })
      if (res.meta.status !== 200) {
        return this.$message.error('设置失败')
      }
      this.$message.success('设置成功')
      // 重新 渲染列表
      this.getRolesList()
      // 关闭 弹窗
      this.editRightsDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped >
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom{
  border-bottom: 1px solid #eee;
}
// 纵向上的居中对齐
.vcenter {
  display: flex;
  align-items: center;
}
</style>
