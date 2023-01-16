<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品添加</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 警告区 -->
      <el-row>
        <el-alert
          title="添加商品信息"
          type="info"
          center
          show-icon
          :closable="false"
        >
        </el-alert>
      </el-row>
      <!-- 进度条区 -->
      <el-row>
        <el-steps
          :space="200"
          :active="activeIndex - 0"
          finish-status="success"
          align-center
        >
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>
      </el-row>
      <!-- tag 栏 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="ruleForm"
        label-width="100px"
      >
        <el-tabs
          v-model="activeIndex"
          :tab-position="'left'"
          :before-leave="beforeTabLeave"
          @tab-click="tabClick"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name" width></el-input>
            </el-form-item>
            <el-form-item label="价格" prop="goods_price">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="数量" prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <!-- 级联选择器 -->
              <el-cascader
                expand-trigger="hover"
                v-model="addForm.goods_cat"
                :options="catesList"
                :props="cascaderProps"
                @change="cateChanged"
                clearable
              >
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyList"
              :key="item.attr_id"
            >
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  :label="n"
                  v-for="(n, i) in item.attr_vals"
                  :key="i"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyList"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 属性 表示图片要上传到的后台api地址 -->
            <el-upload
              :headers="headerObj"
              :action="upLoadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              list-type="picture"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <!-- 添加商品按钮 -->
            <el-button type="primary" class="btnAdd" @click="addGood"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      activeIndex: '0',
      // 添加商品表单
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
      },
      // 商品添加表单的验证规则
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
      },
      // 商品分类列表
      catesList: [],
      // 级联选择参数 props
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
      },
      // 动态参数列表
      manyList: [],
      // 静态属性列表
      onlyList: [],
      // 上传图片的URL地址
      upLoadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的headers请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token'),
      },
      // 预览图片路径
      previewPath: '',
      // 预览图片对话框
      previewVisible: false,
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      console.log(res.data)
      this.catesList = res.data
      console.log(this.parentCatesList)
    },
    // 级联选择器发生变化时会触发这个函数
    cateChanged() {
      // 监听选中的分类是不是三级的 只要监听选中的数组的长度
      if (this.addForm.goods_cat.length !== 3) {
        // 不符合 将级联选择器的内容写空
        this.addForm.goods_cat = []
        this.manyList = []
        return
      }
      console.log(this.addForm.goods_cat)
    },
    // 在选择新的tab时 先进行判断
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类数据')
        return false
      }
      return true
    },
    // tab 标签被选中
    tabClick() {
      if (this.activeIndex === '1') {
        // 获取分类参数列表
        this.getManyList()
      } else if (this.activeIndex === '2') {
        this.getOnlyList()
      }
    },
    // 根据activeName 获取参数数据
    async getManyList() {
      // 判断 是三级分类 可以获取 属性
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: 'many' } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数数据列表失败')
      }
      console.log(res.data)
      res.data.forEach((element) => {
        // 将attr_vals 可选参数 改成列表的形式
        element.attr_vals = element.attr_vals
          ? element.attr_vals.split(' ')
          : []
        // 添加每个属性各自 的inputVisible &  inputValue
        element.inputVisible = false
        element.inputValue = ''
      })
      this.manyList = res.data
      console.log(this.manyList)
    },
    // 获取静态参数列表
    async getOnlyList() {
      // 判断 是三级分类 可以获取 属性
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: 'only' } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数数据列表失败')
      }
      console.log(res.data)
      this.onlyList = res.data
      console.log(this.onlyList)
    },
    // 处理图片的预览效果
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理移除图片的效果
    handleRemove(file) {
      // 1.获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2.从pics数组中，找到这个图片对应的索引值
      const index = this.addForm.pics.findIndex((x) => x.pic === filePath)
      // 3.调用数组的splice方法，把图片信息对象，从pics数组中移除
      this.addForm.pics.splice(index, 1)
      console.log(this.addForm)
    },
    // 监听图片上传成功的事件
    handleSuccess(response) {
      console.log(response)
      // 1.拼接得到一个图片信息
      const picInfo = {
        pics: response.data.tmp_path,
      }
      // 2.将图片信息对象 ，push到pics数组中
      this.addForm.pics.push(picInfo)
      // 3.打印添加表单信息
      console.log(this.addForm)
    },
    // 添加商品
    addGood() {
      // 1.进行表单验证
      this.$refs.ruleForm.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('请填写好必要的表单项！')
        }
        // 2.执行添加的业务逻辑
        // this.addForm.goods_cat = this.addForm.goods_cat.join(',')
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 3.处理动态参数
        this.manyList.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' '),
          }
          this.addForm.attrs.push(newInfo)
        })
        // 4.处理静态属性
        this.onlyList.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' '),
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)

        // 5.发起请求添加商品
        // 注意其中的商品名称必须唯一
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败！')
        }
        this.$message.success('添加商品成功！')
        this.$router.push('/goods')
      })
    },
  },
  computed: {
    // 选中的商品分类 id
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[this.addForm.goods_cat.length - 1]
      }
      return -1
    },
  },
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>
