<template>
  <el-container class="home-container">
    <!-- 头部 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="logo" />
        <span>通用电商后台管理系统</span>
      </div>
      <el-button @click="logout">退出</el-button>
    </el-header>
    <!-- 主体内容 -->
    <el-container>
      <!-- 左侧导航栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 折叠/展开按钮 -->
        <div class="toggle-button" @click="toggleButton">|||</div>
        <!-- 菜单 -->
        <el-menu
          class="el-menu-vertical-demo"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >
          <!-- collapse折叠，unique-opened手风琴 -->
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/' +subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' +subItem.path)">
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容 -->
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  data() {
    return {
      // 菜单
      menulist: [],
      // 一级菜单图标
      iconsObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      // 被激活的二级菜单链接地址
      activePath: ''
    }
  },
  created() {
    // 组件被创建时获取菜单
    this.getMenuList()
    // 组件被创建时获取路由菜单链接地址
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    },
    toggleButton() {
      this.isCollapse = !this.isCollapse
    },
    // 保存二级菜单的激活状态
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>
<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex; // 弹性布局
  justify-content: space-between; // 左右对齐
  align-items: center; // 上下居中
  color: #fff;
  font-size: 25px;
  background-color: #5464;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #545c64;
}
.el-main {
  background-color: #eaedfdf1;
}
.el-menu {
  border-right: none;
}
.toggle-button {
  background-color: #545a70;
  font-size: 15px;
  line-height: 25px;
  color: #fff;
  text-align: center;
  // |||间距
  letter-spacing: 0.2em;
  // 鼠标移入时显示小手
  cursor: pointer;
}
</style>
