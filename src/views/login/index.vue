<template>
  <div class="login-container">
    <video
      poster="../../assets/images/login/video-cover.jpeg"
      loop
      autoplay
      muted
    >
      <source src="../../assets/images/login/night.mp4">
    </video>
    <el-form
      ref="loginFormRef"
      :model="loginForm"
      :rules="loginRules"
      class="login-form"
      autocomplete="on"
      label-position="left"
    >
      <div class="title-container">
        <h3 class="title">Login Form</h3>
      </div>

      <el-form-item prop="username">
        <span class="svg-container">
          <svg-vue icon="user" />
        </span>
        <el-input
          ref="username"
          v-model="loginForm.username"
          :placeholder="Username"
          name="username"
          type="text"
          tabindex="1"
          autocomplete="on"
        />
      </el-form-item>

      <el-tooltip v-model="capsTooltip" content="Caps lock is On" placement="right" manual>
        <el-form-item prop="password">
          <span class="svg-container">
            <i class="el-icon-lock" />
          </span>
          <el-input
            :key="passwordType"
            v-model="loginForm.password"
            :type="passwordType"
            placeholder="Password"
            name="password"
            tabindex="2"
            autocomplete="on"
            @keyup="checkCapslock"
            @blur="capsTooltip = false"
            @keyup.enter="handleLogin"
          />
          <span class="show-pwd" @click="showPwd">
            <svg-vue :icon="passwordType === 'password' ? 'eye' : 'eye-open'"/>
          </span>
        </el-form-item>
      </el-tooltip>

      <el-button :loading="loading" type="primary" style="width:100%;margin-bottom:30px;"
                 @click.prevent="handleLogin">Login
      </el-button>

      <div style="position:relative">
        <div class="tips">
          <span>Username : admin</span>
          <span>Password : any</span>
        </div>
        <div class="tips">
          <span style="margin-right:18px;">Username : editor</span>
          <span>Password : any</span>
        </div>

        <el-button class="thirdparty-button" type="primary" @click="showDialog=true">
          Or connect with
        </el-button>
      </div>
    </el-form>

    <el-dialog title="Or connect with" v-model="showDialog">
      Can not be simulated on local, so please combine you own business simulation! ! !
      <br>
      <br>
      <br>
      <social-sign/>
    </el-dialog>
  </div>
</template>

<script>
import {defineComponent, ref, unref, isRef, toRefs, onMounted, watchEffect, reactive, nextTick} from "vue";
import {useRoute, useRouter} from "vue-router";
import {validUsername} from '@/utils/validate'
import SocialSign from './components/SocialSignin'
import {useStore} from "vuex";

export default {
  name: 'Login',
  components: {SocialSign},
  data() {
    return {}
  },
  setup(props, ctx) {
    const store = useStore();
    const route = useRoute()
    const router = useRouter()

    const loginFormRef = ref(null)
    const loginForm = reactive({
      username: 'admin',
      password: 'admin'
    })
    const loginRules = {
      username: [{required: true, trigger: 'blur', validator: validateUsername}],
      password: [{required: true, trigger: 'blur', validator: validatePassword}]
    }

    let passwordType = ref('password')
    let capsTooltip = ref(false)
    let loading = ref(false)
    let showDialog = ref(false)
    let redirect = ref(null)
    let otherQuery = reactive({})

    watchEffect(() => {
      const query = route.query
      if (query) {
        redirect = query.redirect
        otherQuery = getOtherQuery(query)
      }
    })

    function getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }

    function validateUsername(rule, value, callback) {
      if (!validUsername(value)) {
        callback(new Error('Please enter the correct user name'))
      } else {
        callback()
      }
    }

    function validatePassword(rule, value, callback) {
      if (value.length < 3) {
        callback(new Error('The password can not be less than 6 digi ts'))
      } else {
        callback()
      }
    }

    function checkCapslock(e) {
      const {key} = e
      capsTooltip = key && key.length === 1 && (key >= 'A' && key <= 'Z')
    }

    function showPwd() {
      if (passwordType.value === 'password') {
        passwordType = ''
      } else {
        passwordType = 'password'
      }
      nextTick(() => {
        loginForm.password.value && loginForm.password.focus()
      })
    }

    const handleLogin = () => {
      const form = unref(loginFormRef);
      form.validate().then((valid) => {
        if (valid) {
          loading = true
          store.dispatch('user/login', loginForm).then(() => {
            router.push({path: redirect || '/', query: otherQuery})
            loading = false
          }).catch((err) => {
            console.log(err)
            loading = false
          })
        } else {
          console.log('error submit!!')
          return false
        }
      }).catch(err => {
        console.log('login valid', err)
      })
    }

    onMounted(() => {
      if (loginForm.username.value === '') {
        loginForm.username.value && loginForm.username.focus()
      } else if (loginForm.password.value === '') {
        loginForm.password.value && loginForm.password.focus()
      }
    })

    return {
      loginRules, loginForm, loginFormRef, checkCapslock, handleLogin, getOtherQuery
      , passwordType, capsTooltip, loading, showDialog, redirect, otherQuery
    }
  },
  created() {
    // window.addEventListener('storage', this.afterQRScan)
  },

  unmounted() {
    // window.removeEventListener('storage', this.afterQRScan)
  }
}
</script>

<style lang="scss">
// Login page
$lightGray: #eee;
$darkGray:#889aa4;
$loginBg: #2d3a4b;
$loginCursorColor: #fff;
// References: https://www.zhangxinxu.com/wordpress/2018/01/css-caret-color-first-line/
@supports (-webkit-mask: none) and (not (cater-color: $loginCursorColor)) {
  .login-container .el-input {
    input {
      color: $loginCursorColor;
    }
    input::first-line {
      color: $lightGray;
    }
  }
}
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      height: 47px;
      background: transparent;
      border: 0px;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $lightGray;
      caret-color: $loginCursorColor;
      -webkit-appearance: none;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px #543d43 inset !important;
        -webkit-text-fill-color: #fff !important;
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
// Login page
$lightGray: #eee;
$darkGray:#889aa4;
$loginBg: #2d3a4b;
$loginCursorColor: #fff;
.login-container {
  height: 100%;
  width: 100%;
  overflow: hidden;
  // background-color: $loginBg;
  video {
    position: absolute;
    /* Vertical and Horizontal center*/
    top: 0; left: 0; right: 0; bottom: 0;
    width:100%;
    height:100%;
    object-fit: cover;
    z-index: -1;
  }
  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
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
    color: $darkGray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $lightGray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }

    .set-language {
      color: #fff;
      position: absolute;
      top: 3px;
      font-size: 18px;
      right: 0px;
      cursor: pointer;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $darkGray;
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
</style>
