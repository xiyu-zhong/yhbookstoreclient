<template>
  <div class="login-container" :style="{backgroundImage: 'url(' + Bgimg + ')' }">
    <!--登录面板内容部分-->
    <div class="login-inner">
      <!--面板头部-->
      <div class="login-header">
        <div class="login-logo">
          <img src="./images/logo-round.png" alt width="150" />
        </div>
        <!--面板标题-->
      </div>
      <!--面板表单部分-->
      <div class="login-content">
        <form>
          <!--账号登录部分-->
          <div :class="{current: !loginMode}">
            <section>
              <section class="login-message">
                <input type="text" maxlength="11" placeholder="账号" v-model="user_name" />
              </section>
              <section class="login-verification">
                <input type="password" maxlength="18" placeholder="密码" v-if="pwdMode" v-model="pwd" />
                <input type="text" maxlength="18" placeholder="密码" v-else v-model="pwd" />
                <div class="switch-show">
                  <img
                    @click.prevent="dealPwdMode(false)"
                    :class="{on: pwdMode}"
                    src="./images/hide_pwd.png"
                    alt
                    width="20"
                  />
                  <img
                    @click.prevent="dealPwdMode(true)"
                    :class="{on: !pwdMode}"
                    src="./images/show_pwd.png"
                    alt
                    width="20"
                  />
                </div>
              </section>
              <section class="login-message">
                <input type="text" maxlength="11" placeholder="验证码" v-model="captcha" />
                <img
                  ref="captcha"
                  class="get-verification"
                  src="http://localhost:3000/api/captcha"
                  alt="captcha"
                  @click.prevent="getCaptcha()"
                />
              </section>
              <section class="login-hint">没有注册过的话，就直接帮您注册了啊</section>
            </section>
          </div>
          <button class="login-submit" @click.prevent="login()">登录</button>
        </form>
        <button class="login-back" @click="$router.back()">返回</button>
      </div>
    </div>
  </div>
</template>

<script>
import { getPhoneCode, phoneCodeLogin, pwdLogin } from "./../../api/index";
import { MessageBox } from "element-ui";
import { mapActions } from "vuex";
import bgimg from "./images/bg.png";
export default {
  name: "Login",
  data() {
    return {
      Bgimg: bgimg,
      loginMode: false, // 登录方式, true 验证码登录 false 账号登录
      phone: "", // 手机号码
      countDown: 0, // 倒计时
      pwdMode: true, // 密码的显示方式 true 密文 false 明文
      pwd: "", // 密码
      code: "", // 验证码
      userInfo: {}, // 用户的信息
      user_name: "", // 用户名
      captcha: "" // 图形验证码
    };
  },
  computed: {
    // 验证手机号是否合理
    phoneRight() {
      return /^[1][3,4,5,7,8][0-9]{9}$/.test(this.phone);
    }
  },
  methods: {
    ...mapActions(["syncUserInfo"]),
    // 1. 登录的模式
    dealLoginMode(flag) {
      this.loginMode = flag;
    },
    // 2. 获取短信验证码
    async getVerifyCode() {
      // 2.1 开启倒计时
      if (this.phoneRight) {
        this.countDown = 60;
        this.intervalId = setInterval(() => {
          this.countDown--;
          // 判断
          if (this.countDown === 0) {
            clearInterval(this.intervalId);
          }
        }, 1000);
      }

      // 2.2 获取短信验证码
      let result = await getPhoneCode(this.phone);
      console.log(result);

      // 2.3 获取验证码失败
      if (result.err_code === 0) {
        // 提示信息
        MessageBox({
          type: "info",
          message: result.message,
          showClose: true
        });

        // 其他处理
        /*
          clearInterval(this.intervalId);
          this.countDown = 0;
          */
      } else if (result.success_code === 200) {
        MessageBox({
          type: "info",
          message: `验证码为:${result.message}, 短信功能暂未开通,十分抱歉对您产生的不便`,
          showClose: true
        });
      }
    },
    // 3. 密码的显示方式
    dealPwdMode(flag) {
      this.pwdMode = flag;
    },

    async login() {
      // 4.1 登录模式
      if (this.loginMode) {
        // 验证码登录
        // 4.2 前台校验
        if (!this.phone) {
          MessageBox({
            type: "info",
            message: "请输入手机号码!",
            showClose: true
          });
          return;
        } else if (!this.phoneRight) {
          MessageBox({
            type: "info",
            message: "请输入正确手机号码!",
            showClose: true
          });
          return;
        }

        if (!this.code) {
          MessageBox({
            type: "info",
            message: "请输入验证码!",
            showClose: true
          });
          return;
        } else if (!/^\d{6}$/gi.test(this.code)) {
          MessageBox({
            type: "info",
            message: "请输入正确的验证码!",
            showClose: true
          });
          return;
        }
        // 4.3 手机验证码登录
        const result = await phoneCodeLogin(this.phone, this.code);
        if (result.success_code === 200) {
          this.userInfo = result.message;
          this.phone = ""; // 手机号码
          this.countDown = 0; // 倒计时
          clearInterval(this.intervalId);
          this.code = ""; // 验证码
          this.pwd = ""; // 密码
          this.user_name = ""; // 用户名
          this.captcha = ""; // 图形验证码
        } else {
          this.userInfo = {
            message: "登录失败, 手机号或验证码不正确!"
          };
        }
      } else {
        // 账号和密码登录
        // 4.4 前端校验
        if (!this.user_name) {
          MessageBox({
            type: "info",
            message: "请输入账号!",
            showClose: true
          });
          return;
        } else if (!this.pwd) {
          MessageBox({
            type: "info",
            message: "请输入密码!",
            showClose: true
          });
          return;
        } else if (!this.captcha) {
          MessageBox({
            type: "info",
            message: "请输入验证码!",
            showClose: true
          });
          return;
        }
        // 4.5 用户名和密码的登录
        const result = await pwdLogin(this.user_name, this.pwd, this.captcha);
        if (result.success_code === 200) {
          this.userInfo = result.message;
          this.phone = ""; // 手机号码
          this.countDown = 0; // 倒计时
          clearInterval(this.intervalId);
          this.code = ""; // 验证码
          this.pwd = ""; // 密码
          this.user_name = ""; // 账号
          this.captcha = ""; // 图形验证码
        } else {
          MessageBox({
            type: "info",
            message: "登录失败, 账号或密码或验证码不正确!",
            showClose: true
          });
        }
      }

      // 6. 后续处理
      if (!this.userInfo.id) {
        // 失败
        MessageBox({
          type: "info",
          message: this.userInfo.message,
          showClose: true
        });
      } else {
        // 成功
        // 6.1 同步用户数据
        this.syncUserInfo(this.userInfo);
        // 6.2 回到主界面
        MessageBox({
          type: "success",
          message: "登录成功!",
          showClose: true
        });
        this.$router.back();
        window.localStorage.setItem("userInfo", JSON.stringify(this.userInfo));
        window.localStorage.removeItem("adminInfo");
      }
    }
  }
};
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '../../common/stylus/mixins.styl';

.login-container {
  width: 100%;
  height: 100%;
  background-repeat: no-repeat;
  background-size: 100% 100%;
  padding-top: 50px;

  .login-inner {
    padding: 30px;
    border-radius: 12px;
    width: 28%;
    margin: 0 auto;
    background: #fff;

    .login-header {
      .login-logo {
        font-size: 20px;
        text-align: center;

        img {
          width: 100%;
        }
      }

      .login-header-title {
        padding-top: 40px;
        padding-bottom: 10px;
        text-align: center;

        > a {
          color: #333;
          font-size: 14px;
          padding-bottom: 4px;

          &:first-child {
            margin-right: 40px;
          }

          &.current {
            color: #337ab7;
            font-weight: 700;
            border-bottom: 2px solid #337ab7;
          }
        }
      }
    }

    .login-content {
      > form {
        > div {
          display: none;

          &.current {
            display: block;
          }

          input {
            width: 100%;
            height: 100%;
            padding-left: 8px;
            box-sizing: border-box;
            border: 1px solid #ddd;
            border-radius: 4px;
            outline: 0;
            font: 400 14px Arial;

            &:focus {
              border: 1px solid #337ab7;
            }
          }

          .login-message {
            position: relative;
            margin-top: 16px;
            height: 48px;
            font-size: 14px;
            background: #fff;
            border-radius: 12px;

            input {
              border-radius: 12px;
            }

            .get-verification {
              position: absolute;
              top: 50%;
              right: 10px;
              transform: translateY(-50%);
              border: 0;
              color: #ccc;
              font-size: 14px;
              background: transparent;

              &.phone_right {
                color: purple;
              }
            }
          }

          .login-verification {
            position: relative;
            margin-top: 16px;
            height: 48px;
            border-radius: 12px;
            font-size: 14px;
            background: #fff;

            input {
              border-radius: 12px;
            }

            .switch-show {
              position: absolute;
              right: 10px;
              top: 12px;

              img {
                display: none;
              }

              img.on {
                display: blockfff;
              }
            }
          }

          .login-hint {
            margin-top: 12px;
            color: #337ab7;
            font-size: 12px;
            line-height: 20px;

            > a {
              color: #337ab7;
            }
          }
        }

        .login-submit {
          display: block;
          width: 100%;
          height: 42px;
          margin-top: 30px;
          border-radius: 4px;
          background: #337ab7;
          color: #fff;
          text-align: center;
          font-size: 16px;
          line-height: 42px;
          border: 0;
        }
      }

      .login-back {
        display: block;
        width: 100%;
        height: 42px;
        margin-top: 15px;
        border-radius: 4px;
        background: transparent;
        border: 1px solid #337ab7;
        color: #337ab7;
        text-align: center;
        font-size: 16px;
        line-height: 42px;
      }
    }
  }
}
</style>
