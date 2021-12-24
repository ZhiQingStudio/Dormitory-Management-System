<template>
  <div class="login-container">
    <div id="login-background">
      <!-- 登录表单 -->
      <!-- Element UI的el-form Form 表单 -->
      <!-- 由输入框、选择器、单选框、多选框等控件组成，用以收集、校验、提交数据 -->
      <!-- label-position 属性改变表单域标签的位置 -->
      <!-- rules 属性传入表单验证规则 -->
      <!-- model 属性传入表单数据对象  -->
      <!-- autocomplete 属性传入是否应该启用自动完成功能 -->
      <!-- ref用来获取DOM节点 -->
      <el-form
        ref="loginForm"
        :model="loginForm"
        :rules="loginRules"
        class="login-form"
        autocomplete="on"
        label-position="left"
      >
        <div id="login_center">
          <div id="login_left">
            <div id="login_left_context" class="phone_not">
              <h3>欢迎使用宿舍管理系统</h3>
              <p>简单，快捷，让你的日常生活更加简单</p>
              <p>感谢使用</p>
            </div>
            <p class="login_context_footer">
              <span>©2021 ZhiQingStudio. All rights reserved.</span>
            </p>
          </div>

          <div class="login_right">
            <div id="login_view">
              <h4>登录界面</h4>

              <div id="login_form">
                <p>账号</p>
                <!-- 学号/职工号 -->
                <!-- prop设置或返回被选元素的属性和值 -->
                <el-form-item prop="account">
                  <span class="svg-container">
                    <svg-icon icon-class="user" />
                  </span>
                  <el-input
                    ref="account"
                    v-model="loginForm.account"
                    placeholder="学号/职工号"
                    name="account"
                    type="text"
                    tabindex="1"
                    autocomplete="on"
                  />
                </el-form-item>
                <!-- 学号/职工号 -->

                <p>密码</p>

                <!-- 密码 -->
                <el-tooltip
                  v-model="capsTooltip"
                  content="大写锁定键已开启"
                  placement="right"
                  manual
                >
                  <el-form-item prop="password">
                    <span class="svg-container">
                      <svg-icon icon-class="password" />
                    </span>
                    <el-input
                      :key="passwordType"
                      ref="password"
                      v-model="loginForm.password"
                      :type="passwordType"
                      placeholder="密码"
                      name="password"
                      tabindex="2"
                      autocomplete="on"
                      @keyup.native="checkCapslock"
                      @blur="capsTooltip = false"
                      @keyup.enter.native="handleLogin"
                    />
                    <span class="show-pwd" @click="showPwd">
                      <svg-icon
                        :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"
                      />
                    </span>
                  </el-form-item>
                </el-tooltip>
                <!-- 密码 -->

                <!-- 确认密码 -->
                <el-form-item v-if="registerMode" prop="repassword">
                  <span class="svg-container">
                    <svg-icon icon-class="user" />
                  </span>
                  <el-input
                    ref="repassword"
                    v-model="loginForm.repassword"
                    placeholder="确认密码"
                    name="repassword"
                    type="password"
                    tabindex="1"
                    autocomplete="on"
                  />
                </el-form-item>
                <!-- 确认密码 -->

                <!-- 登录按钮 -->
                <el-button
                  v-if="!registerMode"
                  :loading="loading"
                  type="primary"
                  style="width: 100%; margin-bottom: 30px"
                  @click.native.prevent="handleLogin"
                  >登录</el-button
                >
                <!-- 登录按钮 -->

                <!-- 注册按钮 -->
                <el-button
                  v-if="registerMode"
                  :loading="loading"
                  type="primary"
                  style="width: 100%; margin-bottom: 30px; margin-left: 0;"
                  @click.native.prevent="handleRegister"
                  >注册</el-button
                >
                <!-- 注册按钮 -->

                <!-- 切换为注册 -->
                <a class="toggle" @click="registerMode = !registerMode">
                  <span v-if="!registerMode">学生注册</span>
                  <span v-else>返回登录</span>
                </a>
                <!-- 切换为注册 -->
              </div>
              <p class="login_context_footer phone">
                <span>©2021 ZhiQingStudio. All rights reserved.</span>
              </p>
            </div>
          </div>
        </div>
      </el-form>
      <!-- 登录表单 -->
    </div>
  </div>
</template>

<script>
import { register } from '@/api/user'
export default {
  name: 'Login',
  data() {
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('密码需要6位以上'))
      } else {
        callback()
      }
    }
    const validatorRepassword = (rule, value, callback) => {
      if (value !== this.loginForm.password && this.registerMode) {
        callback(new Error('两次密码不符合'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        account: '',
        password: '',
        repassword: ''
      },
      loginRules: {
        account: [{ required: true, trigger: 'blur' }],
        password: [
          { required: true, trigger: 'blur', validator: validatePassword }
        ],
        repassword: [{ validator: validatorRepassword, trigger: 'blur' }]
      },
      passwordType: 'password',
      registerMode: false,
      capsTooltip: false,
      loading: false,
      redirect: undefined, // redirect 负责获取用户被跳转到登录页面前的页面，可以让用户登录后直接访问到想要访问的页面
      otherQuery: {}
    }
  },
  watch: {
    $route: {
      // 路由重定向
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  mounted() {
    // 表单获取焦点
    if (this.loginForm.account === '') {
      this.$refs.account.focus()
    } else if (this.loginForm.password === '') {
      this.$refs.password.focus()
    }
  },
  methods: {
    // 检查键盘大写锁定键
    checkCapslock(e) {
      const { key } = e
      this.capsTooltip = key && key.length === 1 && key >= 'A' && key <= 'Z'
    },
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate((valid) => {
        if (valid) {
          this.loading = true
          this.$store
            .dispatch('user/login', this.loginForm)
            .then(() => {
              this.$router.push({
                path: this.redirect || '/',
                query: this.otherQuery
              })
              this.loading = false
            })
            .catch(() => {
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    handleRegister() {
      this.$refs.loginForm.validate((valid) => {
        if (valid) {
          this.loading = true
          register({
            account: this.loginForm.account,
            password: this.loginForm.password
          }).then(() => {
            this.$message({ message: '注册成功', type: 'success' })
            this.loading = false
            this.registerMode = false
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }
  }
}
</script>

<style lang="scss">
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
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $bg;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        // box-shadow: 0 0 0px 1000px $bg inset !important;
        // -webkit-text-fill-color: $cursor !important;
        -webkit-text-fill-color: #283443;
        transition: background-color 50000s ease-in-out 0s;
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

.login-container {
  font-family: '微软雅黑', '幼圆', 'microsoft yahei';
  min-height: 100%;
  width: 100%;
  position: fixed;
  background-image: url(https://raw.githubusercontent.com/Clarice0997/Dormitory-Management-System/main/Page%20Design/Img/bg.jpg);
  background-repeat: no-repeat;
  background-size: cover;
  overflow: hidden;

  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
    .toggle {
      width: 100%;
      display: block;
      font-size: 14px;
      text-align: center;
      color: #606266;
      margin-top: -10px;
    }
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
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

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }
}

#login_left {
  height: 292px;
  width: 740px;
  background-color: rgba(81, 136, 108, 0.8);
  /* 绝对定位 */
  position: absolute;
  top: 26%;
  right: 10%;
  box-shadow: 0 0 30px rgba(81, 136, 108, 0.8);
}

#login_left_context {
  position: absolute;
  padding: 12px 16px;
  color: #fff;
}

.login_context_footer {
  color: #666;
  position: absolute;
  bottom: 8px;
  left: 16px;
}

.login_right {
  /* 不透明度0.9 */
  opacity: 0.9;
  height: 450px;
  width: 350px;
  min-height: 420px;
  background-color: #fff;
  position: absolute;
  right: 12%;
  top: 18%;
  box-shadow: 0 0 30px rgba(81, 136, 108, 0.8);
}

#login_center {
  position: fixed;
  top: 0px;
  bottom: 0px;
  left: 0px;
  right: 0px;
}

#login_view {
  padding: 24px;
  padding-top: 0;
}

#login_view h4 {
  font-size: 28px;
  color: #7fb2de;
  margin-top: 20px;
  margin-bottom: 20px;
}

#login_form {
  margin-top: 20px;
}

#login_form>p{
  margin: 5px;
}

.login_forge {
  margin-top: 10px;
  margin-left: 6px;
  display: block;
  float: right;
}

.linput {
  width: 100%;
  padding: 6px 10px;
  border-width: 1px;
  border-color: #666;
  border-color: rgba(0, 0, 0, 0.6);
  height: 36px;
  outline: none;
  border-style: solid;
  background-color: rgba(255, 255, 255, 0.4);
  /* 修改风格 */
  -webkit-appearance: textfield;
  /* 字距 */
  letter-spacing: normal;
  word-spacing: normal;
  /* 首行缩进 */
  text-indent: 0px;
  margin-bottom: 10px;
}

.lbutton {
  border-color: #0067b8;
  background-color: #0067b8;
  color: #fff;
  width: 100%;
  height: 36px;
  border-width: 1px;
  display: inline-block;
  min-width: 100px;
  padding: 4px 12px 4px 12px;
  margin-top: 4px;
  text-align: center;
  /* 规定段落中的文本不进行换行 */
  white-space: nowrap;
  overflow: hidden;
  /* 	显示省略符号来代表被修剪的文本 */
  text-overflow: ellipsis;
  touch-action: manipulation;
  animation: ActionFadeIn 1s;
  animation-fill-mode: forwards;
}

.lbutton:hover {
  background-color: #005ea7;
}

/* 手机适配 */
.phone {
  display: none;
}

@media (max-width: 840px) {
  #mlogin_left {
    width: 100%;
    height: 100%;
    top: 0;
    right: 0;
  }
  #mlogin_right {
    width: 100%;
    height: 100%;
    top: 0;
    right: 0;
  }
  .mlogin_context_footer {
    left: 24px;
  }
  .phone_not {
    display: none;
  }
  .phone {
    display: block;
  }
}
</style>