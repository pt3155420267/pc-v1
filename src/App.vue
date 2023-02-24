<template>
  <div id="app">
    <nav-header v-if="!this.$route.meta.hideHeader"></nav-header>

    <Login-Dialog
      :dialogType="loginDialogType"
      :status="loginDialogStatus"
      :mobile="dialogMobile"
      :notCancel="cancelStatus"
      @hideLoginDialog="hideLoginDialog"
      @changeType="changeType"
    ></Login-Dialog>

    <bind-new-mobile
      :status="bindNewmobileVisible"
      :active="cancelStatus"
      scene="mobile_bind"
      @cancel="cancelDialog"
    ></bind-new-mobile>

    <code-login-bind-mobile
      :status="codebindmobileVisible"
      :active="cancelStatus"
      scene="mobile_bind"
      @cancel="cancelDialog"
    >
    </code-login-bind-mobile>

    <tencent-face-check
      :status="faceCheckVisible"
      @cancel="cancelFaceCheckDialog"
      @change="faceChecksuccess"
    >
    </tencent-face-check>

    <template v-if="initComplete">
      <keep-alive>
        <router-view v-if="config && this.$route.meta.keepAlive"></router-view>
      </keep-alive>
      <router-view v-if="config && !this.$route.meta.keepAlive"></router-view>
    </template>

    <back-top v-show="backTopStatus"></back-top>

    <sign
      v-if="this.$route.meta.sign && isLogin && configFunc.daySignIn"
    ></sign>
  </div>
</template>
<script>
const _ = require("lodash");

import { mapState, mapMutations } from "vuex";
import NavHeader from "./components/navheader.vue";
import LoginDialog from "./components/logindialog.vue";
import BackTop from "./components/back-top.vue";
import Sign from "./components/sign.vue";
import BindNewMobile from "./views/member/components/bind-new-mobile.vue";
import CodeLoginBindMobile from "./components/code-login-bind-mobile.vue";
import TencentFaceCheck from "./components/tencent-face-check.vue";

export default {
  components: {
    NavHeader,
    LoginDialog,
    BackTop,
    Sign,
    BindNewMobile,
    CodeLoginBindMobile,
    TencentFaceCheck,
  },
  data() {
    return {
      cancelStatus: false,
      backTopStatus: false,
      initComplete: false,
      bindNewmobileVisible: false,
      codebindmobileVisible: false,
      faceCheckVisible: false,
    };
  },
  watch: {
    isLogin(val) {
      if (val) {
        this.msvBind();
      }
    },
   
  },
  computed: {
    ...mapState([
      "loginDialogStatus",
      "loginDialogType",
      "dialogMobile",
      "config",
      "isLogin",
      "configFunc",
      "user",
    ]),
  },
  mounted() {
    this.$router.onReady(() => {
      // msv分销id记录
      if (this.$route.query.msv) {
        this.$utils.saveMsv(this.$route.query.msv);
      }

      // 社交登录回调处理
      if (
        this.$route.query.login_code &&
        this.$route.query.action === "login"
      ) {
        this.CodeLogin(this.$route.query.login_code);
      }

      // 自动登录
      this.getConfig();
      if (this.$utils.getToken()) {
        this.getUser();
      }

      // 初始化完成
      this.initComplete = true;

      // 关闭加载框
      let loadingBoxDom = window.document.getElementById("meedu-loading-box");
      if (loadingBoxDom) {
        loadingBoxDom.remove();
      }
    });

    window.addEventListener("scroll", this.getHeight, true);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.getHeight, true);
  },
  methods: {
    ...mapMutations([
      "loginHandle",
      "hideLoginDialog",
      "changeDialogType",
      "setConfig",
      "showLoginDialog",
      "updateFuncConfig",
    ]),
    cancelDialog() {
      this.codebindmobileVisible = false;
      this.bindNewmobileVisible = false;
    },
    cancelFaceCheckDialog() {
      this.faceCheckVisible = false;
    },
    faceChecksuccess() {
      this.faceCheckVisible = false;
      this.getUser();
    },
    changeType(val) {
      this.changeDialogType(val);
    },
    getHeight() {
      let scrollTop =
        window.pageYOffset ||
        document.documentElement.scrollTop ||
        document.body.scrollTop;

      this.backTopStatus = scrollTop >= 2000;
    },
    msvBind() {
      let msv = this.$utils.getMsv();
      if (!msv) {
        return;
      }

      this.$api.MultiLevelShare.Bind({ msv: msv })
        .then(() => {
          this.$utils.clearMsv();
        })
        .catch((e) => {
          console.log(e.message);
          this.$utils.clearMsv();
        });
    },
    CodeLogin(code) {
      if (this.$utils.getSessionLoginCode(code)) {
        return;
      }
      this.$utils.saveSessionLoginCode(code);
      this.$api.Auth.CodeLogin({ code: code, msv: this.$utils.getMsv() })
        .then((res) => {
          if (res.data.success === 1) {
            this.$utils.saveToken(res.data.token);
            this.getUser();
          } else {
            if (res.data.action === "bind_mobile") {
              this.$utils.saveLoginCode(code);
              this.codebindmobileVisible = true;
              this.cancelStatus = true;
            }
          }
        })
        .catch((e) => {
          this.$message.error(e.message);
        });
    },
    async getUser() {
      try {
        let res = await this.$api.User.Detail();
        this.loginHandle(res.data);

        // 强制绑定用户名
        if (
          this.config &&
          res.data.is_bind_mobile === 0 &&
          this.config.member.enabled_mobile_bind_alert === 1
        ) {
          this.bindNewmobileVisible = true;
          this.cancelStatus = true;
        }
        //强制实名认证
        if (
          this.config &&
          res.data.is_face_verify === false &&
          this.config.member.enabled_face_verify === true
        ) {
          this.faceCheckVisible = true;
        }
      } catch (e) {
        this.$message.error(e.message);
      }
    },
    async getConfig() {
      let res = await this.$api.Other.Config();
      let config = res.data;

      // 保存全局配置
      this.setConfig(config);

      // 插件功能开启/关闭
      let funcTable = this.configFunc;
      funcTable["live"] = _.indexOf(config.enabled_addons, "Zhibo") !== -1;
      funcTable["book"] = _.indexOf(config.enabled_addons, "MeeduBooks") !== -1;
      funcTable["topic"] =
        _.indexOf(config.enabled_addons, "MeeduTopics") !== -1;
      funcTable["paper"] = _.indexOf(config.enabled_addons, "Paper") !== -1;
      funcTable["practice"] = _.indexOf(config.enabled_addons, "Paper") !== -1;
      funcTable["mockPaper"] = _.indexOf(config.enabled_addons, "Paper") !== -1;
      funcTable["wrongBook"] = _.indexOf(config.enabled_addons, "Paper") !== -1;
      funcTable["wenda"] = _.indexOf(config.enabled_addons, "Wenda") !== -1;
      funcTable["share"] =
        _.indexOf(config.enabled_addons, "MultiLevelShare") !== -1;
      funcTable["codeExchanger"] =
        _.indexOf(config.enabled_addons, "CodeExchanger") !== -1;
      funcTable["snapshort"] =
        _.indexOf(config.enabled_addons, "Snapshot") !== -1;
      funcTable["ke"] = _.indexOf(config.enabled_addons, "XiaoBanKe") !== -1;
      funcTable["promoCode"] =
        _.indexOf(config.enabled_addons, "MultiLevelShare") === -1;
      funcTable["daySignIn"] =
        _.indexOf(config.enabled_addons, "DaySignIn") !== -1;
      funcTable["credit1Mall"] =
        _.indexOf(config.enabled_addons, "Credit1Mall") !== -1;
      funcTable["tuangou"] = _.indexOf(config.enabled_addons, "TuanGou") !== -1;
      funcTable["miaosha"] = _.indexOf(config.enabled_addons, "MiaoSha") !== -1;
      funcTable["cert"] = _.indexOf(config.enabled_addons, "Cert") !== -1;
      this.updateFuncConfig(funcTable);

      // 手机设备访问PC站点自动跳转到H5端口地址
      if (this.$utils.isMobile() && config.h5_url) {
        let url = config.h5_url;
        if (this.$route.query.msv) {
          //如果存在msv的话则跳转携带上msv参数
          url = this.$utils.SPAUrlAppend(
            config.h5_url,
            "msv=" + this.$route.query.msv
          );
        }
        window.location.href = url;
      }
    },
  },
};
</script>
