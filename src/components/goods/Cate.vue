<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
       <!-- 添加商品按钮 -->
       <el-row>
        <el-col :span="4">
          <el-button type="primary" @click="addCateDialogShow">添加商品</el-button
          >
        </el-col>
      </el-row>
      <!-- 表格数据区 -->
      <tree-table
      :data="goodsList"
      :columns="columns"
      show-index
      index-text="#"
      :expand-type="false"
      :selection-type="false"
      border>
        <!-- 是否有效插槽 -->
        <template slot="isOk" slot-scope="scope">
          <i class="el-icon-error" v-if="scope.row.cat_deleted" style="color:lightcoral"></i>
          <i class="el-icon-success" v-else style="color:lightgreen"></i>
        </template>
        <!-- 排序 -->
        <template slot="level" slot-scope="scope">
          <el-button  plain disabled size="mini" type="primary" v-if="scope.row.cat_level === 0">一级</el-button>
          <el-button  plain disabled size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-button>
          <el-button  plain disabled size="mini" type="warning" v-else>三级</el-button>
        </template>
        <!-- 操作 -->
        <template slot="setting" slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @closed="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateRef" label-width="100px">
         <!-- 分类名称的表单区 -->
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <!-- 父级分类区 -->
        <el-form-item label="父级分类">
          <!-- 级联选择器 -->
          <el-cascader
            expand-trigger="hover"
            v-model="selectedKey"
            :options="parentCatesList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 记录商品总数
      total: 0,
      // 查询商品参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品数组
      goodsList: [],
      // tree-table 的 columns 信息区
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
          width: '400px'
        },
        {
          label: '是否有效',
          prop: 'cat_deleted',
          type: 'template',
          template: 'isOk'
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
          template: 'setting'
        }
      ],
      // 添加分类对话框的显示
      addCateDialogVisible: false,
      // 添加分类
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      // 添加分类的规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类数据列表 在点击添加分类时更新
      parentCatesList: [],
      // 级联选择参数 props
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 级联选择器绑定的值
      selectedKey: []
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    // 获取商品列表数据
    async getGoodsList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      console.log(res.data)
      this.total = res.data.total
      this.goodsList = res.data.result
      console.log(this.goodsList)
      console.log(this.total)
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsList()
    },
    // 展示添加分类的对话框
    addCateDialogShow() {
      this.getParentCatesList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类数据列表
    async getParentCatesList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类列表失败')
      }
      console.log(res.data)
      this.parentCatesList = res.data
      console.log(this.parentCatesList)
    },
    // 选择器发生变化时
    parentCateChanged () {
      console.log(this.selectedKey)
      // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
      // 反之，就说明没有选中任何父级分类
      if (this.selectedKey.length > 0) {
        // 父级分类的Id
        this.addCateForm.cat_pid = this.selectedKey[this.selectedKey.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKey.length
      } else {
        // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 添加分类
    async addCate() {
      this.$refs.addCateRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品分类失败')
        }
        this.getGoodsList()
        this.addCateDialogVisible = false
      }
      )
    },
    // 添加分类对话框关闭 要重置表单
    addCateDialogClosed() {
      this.$refs.addCateRef.resetFields()
      this.selectedKey = []
      this.addCateForm.cat_name = ''
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader{
  width: 100%;
  height: 30%;
}
</style>
