<template>
  <div>
    <el-form
      v-loading="loading"
      element-loading-text="正在登录......"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
      ref="loginForm" :model="loginForm" :rules="rules" class="loginContainer">
      <h3 class="loginTitle">系统登录</h3>
      <el-form-item prop="username">
        <el-input type="text" v-model="loginForm.username" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input type="password" v-model="loginForm.password" placeholder="请输入密码"></el-input>
      </el-form-item>
      <el-form-item prop="code">
        <el-input type="text" v-model="loginForm.code" placeholder="点击图片更换验证码" @keydown.enter.native="submitLogin"
                  style="width: 250px;margin-right: 5px;"></el-input>
        <img :src="captchaUrl" @click="updateCaptcha">
      </el-form-item>
      <el-button type="primary" style="width: 100%" @click="submitLogin">登录</el-button>
    </el-form>
  </div>
</template>
<script>
  export default {
    name: 'Login',
    components: {},
    props: [],
    data() {
      return {
        // 验证码
        captchaUrl:'/captcha?time=' + new Date(),//获取响应码后端接口
        loginForm: {
          username: 'admin',
          password: '123',
          code: '',
        },
        loading: false, // 加载中
        //校验规则，与表单绑定
        rules: {
          username: [{required: true, message: '请输入用户名', trigger: 'blur'}],
          password: [{required: true, message: '请输入密码', trigger: 'blur'}],
          code: [{required: true, message: '请输入验证码', trigger: 'blur'}]
        }
      }
    },
    mounted(){
 
    },
    methods: {
      // 点击刷新验证码
      updateCaptcha() {
      this.captchaUrl="/captcha?time="+new Date();
 
      },
      submitLogin() {
        // 登录
        this.$refs.loginForm.validate((valid) => {
          if (valid) {
            this.loading = true;//准备调登录接口时，出现正在加载
            //第一个参数请求后端的地址，第二个参数，传给后端的数据
            this.postRequest('/login', this.loginForm).then(resp => {
              this.loading = false;//登录成功后关闭
              if (resp) {
                // 存储用户 token 到 sessionStorage
                const tokenStr = resp.obj.tokenHead + resp.obj.token;
                window.sessionStorage.setItem('tokenStr', tokenStr);
                // 跳转到首页
                // this.$router.push('/home') // 路由跳转，可以回退到上一页
                this.$router.replace('/home') // 路径替换，无法回退到上一页
 
                // 页面跳转
                // 拿到用户要跳转的路径
                let path = this.$route.query.redirect;
                // 用户可能输入首页地址或错误地址，让他跳到首页，否则跳转到他输入的地址
                this.$router.replace((path === '/' || path === undefined) ? '/home' : path)
              }
 
            })
          } else {
            this.$message.error('请输入所有字段！');
            return false;
          }
        })
      }
    }
  }
</script>
<style>
  .loginContainer {
    border-radius: 15px;
    background-clip: padding-box;
    /*属性规定背景的绘制区域 背景被裁剪到内边距框。 margin: 180 px auto;*/
    margin: 180px auto;
    width: 350px;
    padding: 15px 35px;
    background: #fff;
    border: 1px solid #eaeaea;
    box-shadow: 0 0 25px #cac6c6;
    /*  X轴偏移量 Y轴偏移量 [阴影模糊半径] [阴影扩展] [阴影颜色] [投影方式]; */
  }
 
  .loginTitle {
    margin: 0 auto 40px auto;
    text-align: center;
  }
 
  .loginRemember {
    text-align: left;
    margin: 0 0 15px 0;
  }
 
  /*验证码*/
  .el-form-item__content {
    display: flex;
    align-items: center;
  }
</style>