<template>
  <section class="main">
    <div class="container">
      <div class="main-body no-bg">
        <div class="widget signin">
          <div class="widget-header">登录</div>
          <div class="widget-content">
            <template v-if="loginMethod.password">
              <div class="field">
                <label class="label">用户名/邮箱</label>
                <div class="control has-icons-left">
                  <input
                    v-model="username"
                    class="input is-success"
                    type="text"
                    placeholder="请输入用户名或邮箱"
                    @keyup.enter="submitLogin"
                  />
                  <span class="icon is-small is-left"
                    ><i class="iconfont icon-username"
                  /></span>
                </div>
              </div>

              <div class="field">
                <label class="label">密码</label>
                <div class="control has-icons-left">
                  <input
                    v-model="password"
                    class="input"
                    type="password"
                    placeholder="请输入密码"
                    @keyup.enter="submitLogin"
                  />
                  <span class="icon is-small is-left"
                    ><i class="iconfont icon-password"
                  /></span>
                </div>
              </div>

              <div class="field">
                <label class="label">验证码</label>
                <div class="control has-icons-left">
                  <div class="field is-horizontal">
                    <div class="field login-captcha-input">
                      <input
                        v-model="captchaCode"
                        class="input"
                        type="text"
                        placeholder="验证码"
                        @keyup.enter="submitLogin"
                      />
                      <span class="icon is-small is-left"
                        ><i class="iconfont icon-captcha"
                      /></span>
                    </div>
                    <div v-if="captchaUrl" class="field login-captcha-img">
                      <a @click="showCaptcha"><img :src="captchaUrl" /></a>
                    </div>
                  </div>
                </div>
              </div>

              <div class="field">
                <button class="button is-success" @click="submitLogin">
                  登录
                </button>
                <nuxt-link class="button is-text" to="/user/signup">
                  没有账号？点击这里去注册&gt;&gt;
                </nuxt-link>
              </div>
            </template>

            <div
              v-if="loginMethod.qq || loginMethod.github || loginMethod.osc"
              class="third-party-line"
            >
              <div class="third-party-title">
                <span>第三方账号登录</span>
              </div>
            </div>

            <div class="third-parties">
              <github-login v-if="loginMethod.github" :ref-url="ref" />
              <osc-login v-if="loginMethod.osc" :ref-url="ref" />
              <qq-login v-if="loginMethod.qq" :ref-url="ref" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  asyncData({ params, query }) {
    return {
      ref: query.ref,
    }
  },
  data() {
    return {
      username: '',
      password: '',
      captchaId: '',
      captchaUrl: '',
      captchaCode: '',
    }
  },
  head() {
    return {
      title: this.$siteTitle('登录'),
    }
  },
  computed: {
    currentUser() {
      return this.$store.state.user.current
    },
    isLogin() {
      return !!this.currentUser
    },
    loginMethod() {
      return this.$store.state.config.config.loginMethod
    },
  },
  mounted() {
    if (this.redirectIfLogined()) {
      return
    }
    this.showCaptcha()
  },
  methods: {
    async submitLogin() {
      try {
        if (!this.username) {
          this.$message.error('请输入用户名或邮箱')
          return
        }
        if (!this.password) {
          this.$message.error('请输入密码')
          return
        }
        if (!this.captchaCode) {
          this.$message.error('请输入验证码')
          return
        }
        const user = await this.$store.dispatch('user/signin', {
          captchaId: this.captchaId,
          captchaCode: this.captchaCode,
          username: this.username,
          password: this.password,
          ref: this.ref,
        })
        if (this.ref) {
          // 跳到登录前
          this.$linkTo(this.ref)
        } else {
          // 跳到个人主页
          this.$linkTo('/user/' + user.id)
        }
      } catch (e) {
        this.$message.error(e.message || e)
        await this.showCaptcha()
      }
    },
    async showCaptcha() {
      try {
        const ret = await this.$axios.get('/api/captcha/request', {
          params: {
            captchaId: this.captchaId || '',
          },
        })
        this.captchaId = ret.captchaId
        this.captchaUrl = ret.captchaUrl
      } catch (e) {
        this.$message.error(e.message || e)
      }
    },
    /**
     * 如果已经登录了，那么直接跳转
     * @returns {boolean}
     */
    redirectIfLogined() {
      if (this.isLogin) {
        const me = this
        this.$msg({
          message: '登录成功',
          onClose() {
            if (me.ref && !me.$isSigninUrl(me.ref)) {
              me.$linkTo(me.ref)
            } else {
              me.$linkTo('/')
            }
          },
        })
        return true
      }
      return false
    },
  },
}
</script>
