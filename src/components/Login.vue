<template>
  <div class="login_container">
  	<div class="login_box">
      <div class="avatar_box">
        <img src="../assets/logo.png"/>
      </div>
      <el-form ref="loginFormRef" :model="loginForm" :rules="rules" class="login_form" label-width="0px">
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <el-form-item  class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
	export default{
    data(){
      return{
        //登录表单的数据绑定对象
        loginForm:{
          username:'admin',
          password:'123456',
        },
        rules:{
          //验证用户名是否合法
          username:[
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          //验证密码是否合法
          password:[
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
          ],
        }
      }
    },
    methods:{
      //重置表单
      resetLoginForm(){
        this.$refs.loginFormRef.resetFields()
      },
      login(){
        const that=this
        this.$refs.loginFormRef.validate(async(valid,object)=>{
          if(!valid) false;
          const {data:res}=await this.$http.post('login',this.loginForm)
          if(res.meta.status!=200){
            return this.$message.error("登录失败")
          }
          this.$message.success("登录成功")
          //将登录成功之后的token保存到客户端中的sessionStorage中
          console.log(res)
          window.sessionStorage.setItem('token',res.data.token)
          that.$router.push('/home')
        })
      },
    }
  };
</script>

<style scoped lang="less">
	.login_container{
    height: 100%;
		background-color: #2b4b6b;
	}

  .login_box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform:translate(-50%,-50%)
  }

  .avatar_box{
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0;
    position: absolute;
    left: 50%;
    transform: translate(-50%,-50%);
    background-color: #fff;
    img{
      background-color: #eee;
      border-radius: 50%;
      width: 100%;
      height: 100%;
    }
  }

.login_form{
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;//超出box的区域了，所以设置size为边框的大小
}

  .btns{
    display: flex;
    justify-content: flex-end;
  }
</style>
