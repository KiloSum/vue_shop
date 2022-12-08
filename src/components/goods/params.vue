<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 头部的警告区 -->
      <el-alert  show-icon :closable="false"
        title="注意： 只允许为第三级 分类设置相关参数！"
        type="warning">
      </el-alert>
      <!-- 选择商品分类 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 级联选择器 -->
          <el-cascader
            expand-trigger="hover"
            v-model="selectedCateKey"
            :options="CatesList"
            :props="cascaderCateProps"
            @change="CateCascaderChanged"
            clearable>
          </el-cascader>
        </el-col>
      </el-row>
      <!-- tab  标签区 -->
      <el-tabs v-model="activeName" @tab-click="tabClick">
        <!-- 动态参数 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数区 -->
          <el-button size="mini" type="primary" :disabled="isAddBtnDisabled" @click="addDialogVisibleShow">添加参数</el-button>
          <!-- 动态参数表 -->
          <el-table
            :data="manyList"
            style="width: 100%"
            border>
            <!-- 动态参数展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(tag,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="removeParamVal(scope.row,i)">
                  {{tag}}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if=" scope.row.inputVisible"
                  v-model=" scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm( scope.row)"
                  @blur="handleInputConfirm( scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" index-text="#"></el-table-column>
            <el-table-column
              prop="attr_name"
              label="参数名称">
            </el-table-column>
            <el-table-column>
              <template slot-scope = "scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editDialogVisibleShow(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性区 -->
          <el-button size="mini" type="primary" :disabled="isAddBtnDisabled" @click="addDialogVisibleShow">添加属性</el-button>
          <!-- 静态属性表 -->
          <el-table
            :data="onlyList"
            style="width: 100%"
            border>
            <!-- 静态属性展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(tag,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="removeParamVal(scope.row,i)">
                  {{tag}}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if=" scope.row.inputVisible"
                  v-model=" scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm( scope.row)"
                  @blur="handleInputConfirm( scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" index-text="#"></el-table-column>
            <el-table-column
              prop="attr_name"
              label="属性名称">
            </el-table-column>
            <el-table-column>
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editDialogVisibleShow(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加属性 对话框 -->
    <el-dialog
      :title="'添加'+title"
      :visible.sync="addDialogVisible"
      width="50%"
      @closed="addDialogVisibleClosed">
      <!-- 参数名验证 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="title+'名'" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑属性 对话框 -->
    <el-dialog
      :title="'编辑'+title"
      :visible.sync="editDialogVisible"
      width="50%"
      @closed="editDialogVisibleClosed">
      <!-- 参数名验证 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="title+'名'" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParam">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类列表
      CatesList: [],
      // 选中的商品分类列表
      selectedCateKey: [],
      // 级联选择器的参数
      cascaderCateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // tab 默认选中的 pane
      activeName: 'many',
      // 动态参数 数据列表
      manyList: [],
      // 静态属性 数据列表
      onlyList: [],
      // 添加属性对话框是否展示
      addDialogVisible: false,
      // 添加表单 提交的内容
      addForm: {
        attr_name: ''
      },
      // 添加属性的验证规则
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 编辑属性对话框是否展示
      editDialogVisible: false,
      // 要编辑的 属性
      editForm: {
        attr_name: ''
      },
      // 编辑属性的规则
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 要修改的属性id
      attr_id: -1
    }
  },
  created () {
    this.getCatesList()
  },
  methods: {
    // 级联选择器内容变化时触发
    CateCascaderChanged() {
      this.getParamsList()
    },
    // 获取商品分类数据列表
    async getCatesList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      // console.log(res.data)
      this.CatesList = res.data
    },
    // tab 标签被点击
    tabClick() {
      console.log(this.activeName)
      this.getParamsList()
    },
    // 根据activeName 获取参数数据
    async getParamsList() {
      // 监听选中的分类是不是三级的 只要监听选中的数组的长度
      if (this.selectedCateKey.length !== 3) {
        // 不符合 将级联选择器的内容写空
        this.selectedCateKey = []
        this.manyList = []
        this.onlyList = []
        return
      }
      console.log(this.selectedCateKey)
      // 判断 是三级分类 可以获取 属性
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数数据列表失败')
      }
      console.log(res.data)
      res.data.forEach(element => {
        // 将attr_vals 可选参数 改成列表的形式
        element.attr_vals = element.attr_vals ? element.attr_vals.split(' ') : []
        // 添加每个属性各自 的inputVisible &  inputValue
        element.inputVisible = false
        element.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyList = res.data
      } else {
        this.onlyList = res.data
      }
    },
    addDialogVisibleShow() {
      this.addDialogVisible = true
    },
    // 监听添加 属性 对话框关闭
    addDialogVisibleClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加属性
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品属性失败')
        }
        this.getParamsList()
        this.addDialogVisible = false
      })
    },
    // 编辑属性对话框展示
    editDialogVisibleShow(attrid) {
      this.editDialogVisible = true
      this.attr_id = attrid
    },
    // 监听编辑属性对话框的关闭
    editDialogVisibleClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 编辑属性
    editParam() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 200) {
          return this.$message.error('编辑商品属性失败')
        }
        this.getParamsList()
        this.editDialogVisible = false
      })
    },
    // 删除属性
    async removeParam(attrid) {
      const confirmResult = await this.$confirm('此操作将永久删除该' + this.title + ', 是否继续?', '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      // 用户取消了删除操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }

      // 利用接口 删除属性
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrid}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品属性失败')
      }
      this.getParamsList()
    },
    // 文本框触发的事件
    handleInputConfirm(param) {
      if (param.inputValue.trim().length === 0) {
        param.inputValue = ''
        param.inputVisible = false
        return
      }
      // 将inputValue 加进数组
      param.attr_vals.push(param.inputValue.trim())
      // 提交参数到后台服务器
      this.saveParamVals(param)
    },
    // 提交param的val
    async saveParamVals (param) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${param.attr_id}`,
        { attr_name: param.attr_name, attr_sel: param.attr_sel, attr_vals: param.attr_vals.join(' ') })
      if (res.meta.status !== 200) {
        return this.$message.error('编辑商品属性可选值失败')
      }
      this.getParamsList()
    },
    // 点击添加tag 的tag 触发的事件
    showInput(param) {
      param.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除属性的可选项
    removeParamVal(param, i) {
      param.attr_vals.splice(i, 1)
      // 提交param的val
      this.saveParamVals(param)
    }
  },
  computed: {
    // 添加参数按钮是否显示
    isAddBtnDisabled() {
      if (this.selectedCateKey.length === 3) {
        return false
      }
      return true
    },
    // 选中的商品分类 id
    cateId() {
      if (this.selectedCateKey.length === 3) {
        return this.selectedCateKey[this.selectedCateKey.length - 1]
      }
      return -1
    },
    // 添加属性 对话框的title
    title() {
      if (this.activeName === 'many') {
        return '参数'
      }
      return '属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag{
  margin: 0 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
