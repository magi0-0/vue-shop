<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img>
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409eff"
          :unique-opened="true" :collapse="isCollapse"
          :collapse-transition="false" :router="true" :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!--二级菜单-->
            <el-menu-item :index="'/'+it.path" v-for="it in item.children" 
              :key="it.id" @click="saveNavState('/'+it.path)">
              <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{it.authName}}</span>
            </template>
            </el-menu-item>

          </el-submenu>
          
        </el-menu>
      </el-aside>
      <el-main>
        <!--路由占位符-->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data(){
    return{
      menuList:[],
      iconsObj:{
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao'
      },
      //是否折叠
      isCollapse:false,
      //被激活的链接地址
      activePath:'',
    }
  },
  created(){
    this.getMenuList()
    this.activePath=window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      // window.sessionStorage.removeItem("token");
      this.$router.push("/login");
    },
    async getMenuList(){
      const {data:res}=await this.$http.get('menus')
      if(res.meta.status!=200){
        return this.$message.error(res.meta.msg)
      }
      this.menuList=res.data
      console.log(res)
    },
    toggleCollapse(){
      this.isCollapse=!this.isCollapse
    },
    //保存链接的激活状态
    saveNavState(activePath){
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath=activePath
    },
  }
};
</script>

<style lang="less" scoped>
.el-menu{
  border-right:none
}
.iconfont{
  margin-right: 10px;
}
.home-container{
  height: 100%;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;//实现按钮居中对齐
  color: #fff;//文本颜色
  font-size: 20px;
  >div{
    display: flex;
    align-items: center;
    >span{
      margin-left: 5px;
    }
  }
}
.el-aside{
  background-color: #333744;
}
.el-main{
  background-color: #eaedf1;
}
.toggle-button{
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;//字之间的间隔
  cursor: pointer;//鼠标放上去变小手

}
</style>
