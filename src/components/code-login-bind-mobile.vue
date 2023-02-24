<template>
  <div class="mask" v-if="status">
    <div style="height: 440px" class="dialog-login-box">
      <div class="tabs">
        <div class="item-tab active">请绑定用户名</div>
        <a v-if="active" class="linkTab2" @click="cancel()">取消绑定>></a>
      </div>
      <div class="box">
        <div class="input-item">
          <input
            type="text"
            placeholder="请输入用户名"
            autocomplete="off"
            v-model="messageForm.mobile"
            class="input"
            required
          />
        </div>
        <div class="input-item">
          <input
            type="text"
            placeholder="请输入图形验证码"
            autocomplete="off"
            v-model="messageForm.captcha"
            class="input-short"
            required
          />
          <div class="captcha">
            <img
              class="captcha-img"
              :src="captcha.img"
              mode="widthFix"
              @click="getCaptcha"
            />
          </div>
        </div>
        <div class="input-item">
          <input
            type="text"
            placeholder="请输入手机验证码"
            autocomplete="off"
            v-model="messageForm.sms"
            class="input-short"
            required
          />
          <div class="buttons">
            <span class="send-sms-button" @click="sendSms()">
              <template v-if="sms.loading"> {{ sms.current }}s </template>
              <template v-else>获取验证码</template>
            </span>
          </div>
        </div>

        <div class="btn-box" style="margin-bottom: 0px !important">
          <button type="submit" class="submit" @click="CodeMobileValidate()">
            立即绑定
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { mapState, mapMutations } from "vuex";
export default {
  props: ["status", "scene", "active"],
  data() {
    return {
      interval: null,
      loading: false,
      captcha: {
        key: null,
        img: null,
      },
      messageForm: {
        mobile: null,
        sms: null,
        captcha: null,
        password: null,
      },
      sms: {
        loading: false,
        max: 120,
        current: 0,
      },
    };
  },
  watch: {
    status(val) {
      this.getCaptcha();
    },
  },
  computed: {
    ...mapState(["config"]),
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
  methods: {
    ...mapMutations(["loginHandle"]),
    resetDialog() {
      this.sms.loading = false;
      this.sms.current = 0;
      this.messageForm.mobile = null;
      this.messageForm.sms = null;
      this.messageForm.captcha = null;
      this.messageForm.password = null;
    },
    cancel() {
      this.resetDialog();
      clearInterval(this.interval);
      this.$emit("cancel", true);
    },
    getCaptcha() {
      this.$api.Other.Captcha().then((res) => {
        this.captcha = res.data;
      });
    },
    sendSms() {
      if (this.sms.loading) {
        // 冷却中
        return;
      }
      if (!this.messageForm.mobile) {
        this.$message.error("请输入用户名");
        return;
      }
      if (!this.$utils.isPoneAvailable(this.messageForm.mobile)) {
        this.$message.error("请输入正确的用户名");
        return;
      }
      if (!this.messageForm.captcha) {
        this.$message.error("请输入图形验证码");
        return;
      }
      this.$api.Other.SendSms({
        mobile: this.messageForm.mobile,
        image_key: this.captcha.key,
        image_captcha: this.messageForm.captcha,
        scene: this.scene,
      })
        .then(() => {
          // 发送成功
          this.$message.success("发送成功");
          this.sms.loading = true;
          this.sms.current = this.sms.max;
          this.interval = setInterval(() => {
            if (this.sms.current <= 1) {
              this.sms.loading = false;
              clearInterval(this.interval);
            } else {
              this.sms.current--;
            }
          }, 1000);
        })
        .catch((e) => {
          this.getCaptcha();
          this.$message.error(e.message);
        });
    },
    CodeMobileValidate() {
      if (this.loading) {
        return;
      }
      if (!this.messageForm.sms) {
        this.$message.error("请输入手机验证码");
        return;
      }
      if (!this.messageForm.mobile) {
        this.$message.error("请填写绑定用户名");
        return;
      }
      if (!this.$utils.isPoneAvailable(this.messageForm.mobile)) {
        this.$message.error("请输入正确的用户名");
        return;
      }
      this.loading = true;
      this.$api.Member.CodeBindMobile({
        mobile: this.messageForm.mobile,
        code: this.$utils.getLoginCode(),
        mobile_code: this.messageForm.sms,
        msv: this.$utils.getMsv(),
      })
        .then((res) => {
          this.loading = false;
          this.$message.success("绑定成功");
          this.$utils.clearLoginCode();

          let token = res.data.token;
          this.$utils.saveToken(token);
          this.$api.User.Detail()
            .then((res) => {
              this.loginHandle(res.data);
              this.resetDialog();
              this.redirectHandler();
            })
            .catch((e) => {
              this.$message.error(e.message);
            });
        })
        .catch((e) => {
          this.loading = false;
          this.$message.error(e.message);
        });
    },
    redirectHandler() {
      if (this.$route.name === "login") {
        if (this.$route.query.redirect) {
          this.$router.replace({
            path: this.$route.query.redirect,
          });
        } else {
          this.$router.replace({
            name: "index",
          });
        }
      } else {
        location.reload();
      }
    },
  },
};
</script>
<style lang="less" scoped>
.mask {
  width: 100%;
  height: 100%;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  position: fixed;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;

  .dialog-login-box {
    width: 500px;
    max-height: 620px;
    background: #ffffff;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    box-sizing: border-box;
    padding: 30px;
    animation: scaleBig 0.3s;

    .tabs {
      width: 100%;
      height: 44px;
      display: flex;
      flex-direction: row;
      position: relative;

      .linkTab {
        position: absolute;
        top: 5px;
        right: 54px;
        height: 14px;
        font-size: 14px;
        font-weight: 400;
        color: #3ca7fa;
        line-height: 14px;
        cursor: pointer;
        &:hover {
          opacity: 0.8;
        }
      }
      .linkTab2 {
        position: absolute;
        top: 5px;
        right: 0px;
        height: 14px;
        font-size: 14px;
        font-weight: 400;
        color: #3ca7fa;
        line-height: 14px;
        cursor: pointer;
        &:hover {
          opacity: 0.8;
        }
      }

      .btn-close {
        width: 24px;
        height: 24px;
        position: absolute;
        right: 6px;
        top: 2px;
        cursor: pointer;

        &:hover {
          opacity: 0.8;
          animation: rotate360 1s;
        }
      }
      .item-tab {
        width: auto;
        height: 20px;
        font-size: 20px;
        font-weight: 500;
        color: #666666;
        line-height: 20px;
        margin-right: 50px;
        cursor: pointer;
        position: relative;
        &.active {
          color: #333333;
        }
        .actline {
          width: 80px;
          height: 4px;
          background: #3ca7fa;
          position: absolute;
          left: 0px;
          top: 40px;
        }
      }
    }
    .box {
      width: 100%;
      margin-top: 30px;
      display: flex;
      flex-direction: column;
      .qrode {
        width: 300px;
        height: 300px;
        margin: 0 auto;
        margin-bottom: 30px;
      }
      .box-mobile {
        width: 100%;
        height: 24px;
        display: flex;
        flex-direction: row;
        align-items: center;
        margin-bottom: 50px;
        span {
          height: 18px;
          font-size: 18px;
          font-weight: 600;
          color: #333333;
          line-height: 18px;
          strong {
            height: 24px;
            font-size: 24px;
            line-height: 24px;
          }
        }
      }
      .input-item {
        width: 100%;
        margin-bottom: 30px;
        display: flex;
        flex-direction: row;
        align-items: center;
        .input {
          width: 100%;
          height: 54px;
          background: #f4fafe;
          border-radius: 4px;
          border: 1px solid #e5e5e5;
          padding-left: 20px;
          outline: none;
        }
        .input-short {
          width: 310px;
          height: 54px;
          background: #f4fafe;
          border-radius: 4px;
          border: 1px solid #e5e5e5;
          padding-left: 20px;
          margin-right: 20px;
          outline: none;
        }
        .captcha {
          width: 110px;
          height: 39px;
          cursor: pointer;
          img {
            width: 110px;
            height: 39px;
          }
        }
        .buttons {
          flex: 1;
          display: flex;
          flex-direction: row-reverse;
          .send-sms-button {
            display: inline-block;
            width: auto;
            height: 18px;
            font-size: 18px;
            font-weight: 400;
            color: #3ca7fa;
            line-height: 18px;
            cursor: pointer;
            &:hover {
              opacity: 0.8;
            }
          }
        }
      }
      .btn-box {
        width: 100%;
        margin-bottom: 30px;
        .submit {
          width: 100%;
          height: 54px;
          background: #3ca7fa;
          border-radius: 4px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 16px;
          font-weight: 400;
          color: #ffffff;
          line-height: 16px;
          outline: none;
          &:hover {
            opacity: 0.8;
          }
        }
        .logout {
          width: 100%;
          height: 54px;
          background: #fff;
          border-radius: 4px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 14px;
          font-weight: 400;
          color: #999999;
          line-height: 14px;
          outline: none;
          &:hover {
            opacity: 0.8;
          }
        }
      }
      .others {
        width: 100%;
        margin-top: 50px;
        &.active {
          margin-top: 0px;
        }
        .tit {
          width: 100%;
          text-align: center;
          height: 14px;
          font-size: 14px;
          font-weight: 400;
          color: #999999;
          line-height: 14px;
          margin-bottom: 30px;
        }
        .tab-icon {
          width: 100%;
          display: flex;
          justify-content: center;
          .btn-others {
            margin-right: 64px;
            width: 48px;
            height: 48px;
            cursor: pointer;
            &:hover {
              opacity: 0.8;
            }
            &:last-child {
              margin-right: 0px;
            }
          }
        }
      }
    }
  }
}
</style>
