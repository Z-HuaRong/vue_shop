<template>
  <el-container>
    <!-- 头部 -->
    <el-header>
      <div class="header">
        <img src="../assets/2.png" alt="" />
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 主体 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 折叠图标 -->
        <i class="el-icon-s-unfold" @click="menuCollapse"></i>
        <el-menu
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="$route.path"

        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              ><template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ subItem.authName }}</span>
              </template></el-menu-item
            >
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      isCollapse: false,
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao',
      },
    }
  },
  created() {
    this.getMenuList()
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },

    menuCollapse() {
      this.isCollapse = !this.isCollapse
    },
  },
}
</script>

<style scoped>
.el-container {
  height: 100%;
}

.el-header {
  background-color: #9b9bf1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
}

.header img {
  vertical-align: middle;
  border-radius: 60%;
}
.header span {
  margin-left: 22px;
}

.el-aside {
  background-color: #898b92;
}

.el-menu {
  border-right: 0;
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 12px;
}

.el-icon-s-unfold {
  height: 30px;
  font-size: 26px;
  line-height: 30px;
  padding-left: 20px;
  cursor: pointer;
}
</style>
