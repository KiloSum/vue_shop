<template>
<el-container  class="home-container">
  <!-- 头部区域 -->
  <el-header>
    <div>
      <img src="" alt="">
      <span>系统</span>
    </div>
    <el-button type="info" @click="logout">退出</el-button>
  </el-header>
  <!-- 页面主题区域 -->
  <el-container>
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse ? '64px' : '200px'">
      <!-- 侧边栏 折叠&展开 按钮条 -->
      <div class="toggle-button" @click="toggleCollapse">|||</div>
      <!-- 侧边菜单栏区 -->
      <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409EFF" unique-opened :collapse="isCollapse"
      :collapse-transition="false"
      router
      :default-active = "activePath">
      <!-- 一级菜单 -->
        <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
          <!-- 一级菜单的模板区 -->
          <template slot="title">
            <!-- 一级菜单图标 -->
            <i class="el-icon-menu"></i>
            <!-- 一级文本 -->
            <span>{{item.authName}}</span>
          </template>
          <!-- 二级菜单 -->
          <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/'+subItem.path)">
            <!-- 二级菜单的模板去 -->
            <template slot="title">
              <!-- 二级菜单图标 -->
              <i class="el-icon-menu"></i>
              <!-- 二级文本 -->
              <span>{{subItem.authName}}</span>
            </template>
          </el-menu-item>
        </el-submenu>
    </el-menu>
    </el-aside>
    <!-- 右侧内容主体 -->
    <el-main>
      <!-- 占位符 -->
      <router-view></router-view>
    </el-main>
  </el-container>
</el-container>

</template>

<script>
export default {
  data() {
    return {
      menuList: [],
      iconsObj: {
        '125': 'el-icon-user',
        '103': 'el-icon-box',
        '102': 'el-icon-goods',
        '102': 'el-icon-s-order',
        '145': 'el-icon-data-line'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    // 保存链接的激活态
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    },
    logout () {
      // 清空token
      window.sessionStorage.clear()
      // 重定向到登陆界面
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      console.log(res)
    },
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.toggle-button{
  background-color: #4A5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div{
    display: flex;
    align-items: center;
    span{
      margin-left: 15px;
    }
  }
}
.el-aside{
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
}
.el-main{
  background-color: #eaedf1;
}
</style>
