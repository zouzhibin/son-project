<template>
  <div class="login-container">
    <div class="login-box">
      <!--热门功能-->
      <div class="login-box-content-left">
        <div class="f-recommendCon">
          <ul class="f-recommend col-m-1 clearFix">
            <li v-for="item in hotList">
              <span>
                <svg-icon :icon-class="item.icon"/>
                <p>{{ item.name }}</p>
              </span>
            </li>
          </ul>
          <h3>热门功能</h3>
        </div>
      </div>
      <!--登录功能-->
      <div class="login-box-content-right">
        <el-card class="box-card" shadow="never">
          <!--tab-->
          <el-tabs v-model="loginType">
            <el-tab-pane label="密码登录" name="password" :disabled="loading"></el-tab-pane>
            <el-tab-pane label="指纹登录" name="figer" :disabled="loading"></el-tab-pane>
          </el-tabs>
          <!--login-Form-->
          <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" auto-complete="on" label-position="left">
            <el-form-item prop="username">
              <span class="svg-container"><svg-icon icon-class="user"/></span>
              <el-input ref="username" v-model="loginForm.username" placeholder="账号" name="username" type="text" tabindex="1" auto-complete="on" :disabled="loading"/>
            </el-form-item>
            <el-form-item v-show="loginType!=='password'" style="height: 54px;border: none;text-align: center">
              <img class="finger" src="@/assets/common/figer.png" alt style="height: 55px;width: 55px">
            </el-form-item>
            <el-form-item v-show="loginType==='password'" prop="password">
              <span class="svg-container"><svg-icon icon-class="password"/></span>
              <el-input ref="password" v-model="loginForm.password" @keyup.enter.native="handleLogin" placeholder="密码" :key="passwordType" :type="passwordType" name="password" tabindex="2" auto-complete="on"/>
              <span class="show-pwd" @click="showPwd">
                  <svg-icon :icon-class="passwordType==='password'?'eye':'eye-open'"/>
                </span>
            </el-form-item>
            <div class="login-btns">
              <el-button type="primary" class="login-btn" @click.native.prevent="logins" :disabled="loading">登  录</el-button>
            </div>
          </el-form>
          <div class="bottom">
            <!-- <svg-icon icon-class="horn"/> -->
            <img src="@/assets/common/tip.png" alt="">
            <span>为确保账户安全，尽量使用指纹登录</span>
          </div>
        </el-card>
      </div>
    </div>
    <!--指纹弹框-->
    <figer :dialogVisible.sync="dialogVisible"/>
  </div>
</template>

<script>
import {validUsername} from '@/utils/validate'
import Figer from '@/components/fingerprint'
const validateUsername = (rule, value, callback) => {
  if (!value) {
    callback(new Error('请填写账号'))
  } else {
    callback()
  }
}
const validatePassword = (rule, value, callback) => {
  if (value.length < 6) {
    callback(new Error('至少填写6位密码'))
  } else {
    callback()
  }
}
export default {
  name: 'Loginc',
  data() {
    return {
      loginType:'figer',
      loginForm: {username: '', password: ''},
      loginRules: {
        username: [{required: true, trigger: 'blur', validator: validateUsername}],
        password: [{required: true, trigger: 'blur', validator: validatePassword}]
      },
      loading: false,
      passwordType: 'password',
      redirect: undefined,
      dialogVisible:false,
      hotList:[
        {icon:'put-in',name:'入库管理'},
        {icon:'go-out',name:'出库管理'},
        {icon:'inquiry',name:'查库管理'},
        {icon:'report-form',name:'报表管理'},
        {icon:'settings',name:'配置管理'},
        {icon:'take-over',name:'交接管理'}
      ]
    }
  },
  watch: {
    $route: {
      handler: function (route) {
        this.redirect = route.query && route.query.redirect
      },
      immediate: true
    },
    loginType(nvl,ovl){
      this.$refs.loginForm.clearValidate()
    }
  },
  methods: {
    // 密码显示隐藏
    showPwd() {
      if (this.passwordType === 'password') this.passwordType = ''
      else this.passwordType = 'password'
      this.$nextTick(() => this.$refs.password.focus())
    },
    // 密码登录
    handleLogin() {
      this.$refs.loginForm.validate(async valid => {
        if (valid) {
          this.loading = true
          try {
            await this.$store.dispatch('user/login', this.loginForm)
            // this.$router.push({path:this.redirect || '/'})
            this.$router.push({path: '/'})
          }
          catch (e) {}
          finally {
            this.loading = false
          }
          return false
        }
        console.log('error submit!!')
      })
    },
    // 指纹登录
    async fingerLogin(count=1,outLoop) {
      if(outLoop) return this.loading = false
      if(!this.loginForm.username){
        this.$message.error('请填写账号')
        return this.loading = false
      }

      this.loading = true
      this.dialogVisible = true
      try {
        let data = await this.$store.dispatch('user/fingerLogin',this.loginForm)
        if(data.code==0) this.$router.push({path:this.redirect || '/'})
        if(data.code==20002){//指纹异常
          try {
            await this.$confirm(`指纹校验失败是否继续?`, '提示', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' })
            return setTimeout(()=>this.fingerLogin(),1500)
          }catch (e) {}
        }
        if(data.code==20008){//验证中
          if(count<20) return setTimeout(()=>this.fingerLogin(++count,this.dialogVisible ? false : true),1500)
          this.$message.error('指纹校验超时')
        }
        this.loading = false
        this.dialogVisible = false
      }catch (e) {
        this.loading = false;
        this.dialogVisible = false
      }
    },
    // 登录按钮
    logins(){
      switch (this.loginType) {
        case 'figer':
          this.fingerLogin(1)
          break
        case 'password':
          this.handleLogin()
          break
      }
    }
  },
  components:{Figer}
}
</script>

<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */
$bg: #283443;
$light_gray: #fff;
$cursor: #fff;
@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}
/* reset element-ui css */
.login-container {
  .el-input {
    display: block;
    height: 52px;
    width: 85%;
    float: left;
    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      line-height: 28px;
      height: 100%;
      caret-color: $cursor;
      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }
  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>
<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;
$light_green:#009F88;
$light_green1:#76d2c4;
$color1:#DCDFE6;
$color2:#cef3ed;
$color3: #606266;
@mixin bgBdrColor($bgColor:null,$borderColor:null,$fontColor:null) {
  @if $bgColor{background-color: $bgColor;}
  @if $borderColor{border-color: $borderColor;}
  @if $fontColor{color: $fontColor;}
}
.login-container{
  min-height: 100%;
  width: 100%;
  overflow: hidden;
  background: url(../../assets/common/login-bg.png) no-repeat center;
  background-size:cover;
  .login-box{
    width: 770px;
    height: 390px;
    background: rgba(3,100,175, .22);
    position: absolute;
    padding: 10px;
    overflow: hidden;
    border-radius: 5px;
    right: 200px;
    top: 50%;
    transform: translateY(-50%);
  }
  .login-box-content-left{
    position: relative;
    display: inline-block;
    width: 440px;
    height: 370px;
    overflow: hidden;
    background: white;
  }
  .login-box-content-right{
    position: relative;
    display: inline-block;
    margin-left: 10px;
    width: 300px;
    height: 370px;
    overflow: hidden;
    background: white;
    .login-form {
      margin-top: 20px;
    }
    .svg-container {
      padding: 6px 5px 6px 15px;
      color: $dark_gray;
      vertical-align: middle;
      width: 30px;
      // display: inline-block;
      float: left;
    }
    .show-pwd {
      position: absolute;
      right: 10px;
      top: 7px;
      font-size: 16px;
      color: $dark_gray;
      cursor: pointer;
      user-select: none;
    }
    .el-form-item {
      border: 1px solid $dark_gray;
      background: white;
      ::v-deep .el-input input {
        color: $dark_gray;
        caret-color: black;
      }
      &:focus-within {border-color: #036EB8}
    }
    .login-btns {
      .login-btn {width: 100%;margin-top: 10px;}
      // .login-btn.el-button--default{
      //   @include bgBdrColor(#fc620d, #f84813,white);
      //   &:hover{@include bgBdrColor(#ed5704,#f84813)}
      //   &:focus{@include bgBdrColor(#ed5704,#f84813)}
      //   &:active{@include bgBdrColor(null,$color1)}
      // }
    }
    .disable-bg {@include bgBdrColor(#F5F7FA);}
    ::v-deep .el-input input:-webkit-autofill {
      box-shadow: 0 0 0px 1000px white inset !important;
      -webkit-text-fill-color: #889aa4 !important;
    }
  }
  .el-card {
    border: none;
    .bottom {
      margin-top: 25px;
      line-height: 20px;
      .svg-icon {
        font-size: 13px;
        color: #f84813;
      }
      img {
        width: 18px;
        height: 20px;
        vertical-align: middle;
      }
      span{
        margin-left: 5px;
        font-size: 13px;
        color: #171725;
      }
    }
  }
  //tab 样式
  ::v-deep .el-form-item--small .el-form-item__error{
    padding-top: 4px;
  }
  .el-tabs {
    ::v-deep {
      .el-tabs__nav {
        width: 100%;
      }
      .el-tabs__item {
        width: 50%;
        text-align: center;
        &:not(.is-active) {
          color: #171725;
        }
      }
      .el-tabs__active-bar {
        height: 4px;
        border-radius: 2px;
      }
      .el-tabs__nav-wrap::after {
        height: 4px;
        border-radius: 2px;
      }
    }
  }
  // 快捷功能
  .f-recommendCon {
    & h3 {
      font-size: 16px;
      font-weight: 500;
      color: #171725;
      // padding: 26px 0 33px 60px;
      padding-left: 66px;
    }
    & ul{
      list-style: none;
      overflow: hidden;
      margin-top: 40px;
      & li {
        float: left;
        width: 110px;
        margin-right: 10px;
        text-align: center;
        list-style: none;
        p {
          font-size: 14px;
          color: #171725;
          padding: 9px 0 24px;
        }
        .svg-icon {
          font-size: 52px;
          color: #036EB8;
          transition: all .5s ease-in-out;
          position: relative;
        }
      }
    }
  }
}
</style>
