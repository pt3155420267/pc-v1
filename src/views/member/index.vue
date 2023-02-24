<template>
  <div class="content">
    <div class="mask" v-if="openmask">
      <div style="height:284px" class="dialog-login-box">
        <div class="dialog-tabs">
          <div class="item-tab">解绑账号</div>
          <img
            class="btn-close"
            @click="cancel()"
            src="@/assets/img/commen/icon-close.png"
          />
        </div>
        <div class="box">
          <div class="input-item">
            解绑账号后请立即绑定其他社交账号，不然可能导致无法找回原账号，确认操作？
          </div>
        </div>
        <div class="btn-box">
          <button type="submit" class="submit" @click="submitHandle()">
            确认
          </button>
          <button type="cancel" class="cancel" @click="cancel()">
            取消
          </button>
        </div>
      </div>
    </div>
    <div class="box">
      <nav-member :cid="0"></nav-member>
      <div class="project-box">
        <template v-if="loading">
          <skeletonMember></skeletonMember>
        </template>
        <template v-else-if="user">
          <div class="user-box">
            <div class="avatar">
              <img :src="user.avatar" />
            </div>
            <div class="user-info">
              <div class="user-top">
                <div class="nickname">{{ user.nick_name }}</div>
                <div class="role" v-if="user.role_id !== 0 && user.role">
                  VIP
                </div>
                <!--<div class="normal" v-else>普通会员</div>-->
              </div>
              <div
                class="expiration-time"
                v-if="user.role_id !== 0 && user.role_expired_at"
              >
                会员有效期至{{ user.role_expired_at }}
              </div>
            </div>
            <div class="value-box">
              <div class="item">
                <div class="value">{{ user.credit1 }}</div>
                <div class="name">我的积分</div>
              </div>
              <div class="item">
                <div class="value">{{ user.invite_people_count }}</div>
                <div class="name">成功邀请(人)</div>
              </div>
              <div class="item">
                <div class="value">{{ user.invite_balance }}</div>
                <div class="name">邀请余额(元)</div>
              </div>
            </div>
          </div>
          <div class="user-profile">
            <div class="del-user" @click="destroyUser()">注销账号</div>
            <div class="tabs">
              <div
                class="item-tab"
                v-for="(item, index) in tabs"
                :key="index"
                :class="{ active: currentTab === item.id }"
                @click="tabChange(item.id)"
                :is-scroll="false"
              >
                {{ item.name }}
                <div class="actline" v-if="currentTab === item.id"></div>
              </div>
            </div>
            <div class="project-content" v-if="currentTab === 1">
              <div class="item-line">
                <div class="item-left">
                  <div class="item-name">我的头像</div>
                  <div class="item-avatar">
                    <img class="avatar" :src="user.avatar" />
                    <input
                      class="input-avatar"
                      type="file"
                      accept="image/*"
                      @change="uploadAvatar"
                    />
                  </div>
                </div>
                <div class="item-right">
                  <div class="tip">点击图片修改</div>
                </div>
              </div>
              <div class="item-line">
                <div class="item-left">
                  <div class="item-name">我的昵称</div>
                  <div class="item-value" v-if="!editNickStatus">
                    {{ user.nick_name }}
                  </div>
                  <div class="item-value" v-if="editNickStatus">
                    <input
                      class="input"
                      type="text"
                      placeholder="昵称"
                      v-model="form.nick_name"
                    />
                  </div>
                </div>
                <div class="item-right">
                  <div
                    v-if="user.is_set_nickname === 0 && !editNickStatus"
                    class="btn"
                    @click="showEditNick()"
                  >
                    修改
                  </div>
                  <div
                    v-if="user.is_set_nickname === 0 && editNickStatus"
                    class="btn act"
                    @click="saveEditNick()"
                  >
                    保存
                  </div>
                  <div v-if="user.is_set_nickname === 1" class="btn">
                    已修改
                  </div>
                  <div class="tip" v-if="user.is_set_nickname === 0">
                    （只可修改一次）
                  </div>
                </div>
              </div>
              <div class="item-line">
                <div class="item-left">
                  <div class="item-name">用户名</div>
                  <div class="item-value" v-if="user.is_bind_mobile === 1">
                    {{
                      user.mobile.substr(0, 3) + "****" + user.mobile.substr(7)
                    }}
                  </div>
                </div>
                <div class="item-right">
                  <div
                    class="btn"
                    v-if="user.is_bind_mobile === 1"
                    @click="goChangeMobile()"
                  >
                    换绑用户名
                  </div>
                  <div class="btn" v-else @click="goBindMobile()">
                    绑定用户名
                  </div>
                </div>
              </div>
              <div class="item-line">
                <div class="item-left">
                  <div class="item-name">设置(修改)密码</div>
                </div>
                <div class="item-right">
                  <div class="btn" @click="goChangePassword()">
                    点击设置（修改）密码
                  </div>
                </div>
              </div>
              <div class="item-line" v-if="config.socialites.qq === 1">
                <div class="item-left">
                  <div class="item-name">
                    <img src="../../assets/img/commen/icon-qq.png" />绑定QQ
                  </div>
                  <div class="item-value" v-if="user.is_bind_qq === 1">
                    已绑定
                  </div>
                  <div
                    class="item-btn"
                    v-else-if="user.is_bind_qq === 0"
                    @click="goBindQQ()"
                  >
                    点击绑定
                  </div>
                </div>
                <div class="item-right">
                  <div
                    class="btn"
                    v-if="user.is_bind_qq === 1"
                    @click="cancelBindQQ()"
                  >
                    解绑账号
                  </div>
                </div>
              </div>
              <div class="item-line" v-if="config.socialites.wechat_scan === 1">
                <div class="item-left">
                  <div class="item-name">
                    <img
                      src="../../assets/img/commen/icon-wechat.png"
                    />绑定微信
                  </div>
                  <div class="item-value" v-if="user.is_bind_wechat === 1">
                    已绑定
                  </div>
                  <div
                    class="item-btn"
                    v-else-if="user.is_bind_wechat === 0"
                    @click="goBindWeixin()"
                  >
                    点击绑定
                  </div>
                </div>
                <div class="item-right">
                  <div
                    class="btn"
                    v-if="user.is_bind_wechat === 1"
                    @click="cancelBindWeixin()"
                  >
                    解绑账号
                  </div>
                </div>
              </div>
            </div>
            <div class="project-content" v-if="currentTab === 2">
              <profile></profile>
            </div>
          </div>
        </template>
      </div>
    </div>
    <change-password
      v-if="user"
      :status="changePasswordVisible"
      :mobile="user.mobile"
      scene="password_reset"
      @cancel="cancel"
      @success="success"
    ></change-password>
    <mobile-validate
      v-if="user"
      :status="mobileValidateVisible"
      :mobile="user.mobile"
      scene="mobile_bind"
      @cancel="cancel"
      @success="successMobileValidate"
    ></mobile-validate>
    <bind-mobile
      :status="bindMobileVisible"
      :sign="bindMobileSign"
      scene="mobile_bind"
      @cancel="cancel"
      @success="success"
    ></bind-mobile>
    <bind-new-mobile
      :status="bindNewmobileVisible"
      :active="false"
      scene="mobile_bind"
      @cancel="cancel"
      @success="success"
    ></bind-new-mobile>
    <bind-weixin
      :status="bindWeixinVisible"
      @cancel="cancel"
      @success="success"
    ></bind-weixin>
    <destroy-user :status="destroyUserVisible" @cancel="cancel"></destroy-user>
    <nav-footer></nav-footer>
  </div>
</template>
<script>
import { mapState, mapMutations } from "vuex";
import Utils from "@/js/utils";
import axios from "axios";
import localConfig from "../../js/config";
import NavFooter from "../../components/footer.vue";
import NavMember from "../../components/navmember.vue";
import Profile from "./components/profile.vue";
import ChangePassword from "./components/change-password.vue";
import MobileValidate from "./components/mobile-validate.vue";
import BindWeixin from "./components/bind-weixin.vue";
import BindMobile from "./components/bind-mobile.vue";
import BindNewMobile from "./components/bind-new-mobile.vue";
import DestroyUser from "./components/destroy-user.vue";
import SkeletonMember from "../../components/skeleton/skeletonMember.vue";

// 请求域名
axios.defaults.baseURL = localConfig.url;
axios.defaults.timeout = 10000;

// 请求拦截器(附带上token)
axios.interceptors.request.use(
  (config) => {
    const token = Utils.getToken();
    token && (config.headers.Authorization = "Bearer " + token);
    return config;
  },
  (error) => {
    return Promise.error(error);
  }
);
export default {
  components: {
    NavFooter,
    NavMember,
    Profile,
    ChangePassword,
    MobileValidate,
    BindWeixin,
    BindMobile,
    BindNewMobile,
    DestroyUser,
    SkeletonMember,
  },
  data() {
    return {
      openmask: false,
      user: null,
      editNickStatus: false,
      currentTab: 1,
      loading: false,
      formLoading: false,
      form: {
        nick_name: null,
      },
      tabs: [
        {
          name: "基本信息",
          id: 1,
        },
        {
          name: "实名认证",
          id: 2,
        },
      ],
      changePasswordVisible: false,
      mobileValidateVisible: false,
      bindMobileSign: null,
      bindMobileVisible: false,
      bindNewmobileVisible: false,
      destroyUserVisible: false,
      app: null,
      bindWeixinVisible: false,
    };
  },
  computed: {
    ...mapState(["isLogin", "config"]),
  },
  mounted() {
    this.$router.onReady(() => {
      // 社交绑定回调处理
      if (this.$route.query.login_code && this.$route.query.action === "bind") {
        this.CodeBind(this.$route.query.login_code);
      }
      if (this.$route.query.login_err_msg) {
        this.$message.error(this.$route.query.login_err_msg);
      }
      if (this.user) {
        this.form.nick_name = this.user.nick_name;
      }
      this.getData();
    });
  },
  methods: {
    ...mapMutations(["loginHandle", "setConfig"]),
    showEditNick() {
      this.editNickStatus = true;
    },
    resetData() {
      this.editNickStatus = false;
      this.$api.Member.Detail().then((res) => {
        this.user = res.data;
      });
    },
    tabChange(key) {
      this.currentTab = key;
    },
    goChangeMobile() {
      this.mobileValidateVisible = true;
    },
    goBindMobile() {
      this.bindNewmobileVisible = true;
    },
    goBindQQ() {
      let host = this.$utils.getShareHost() + "member";
      let token = this.$utils.getToken();
      let redirect = encodeURIComponent(host);
      window.location.href =
        this.config.url +
        "/api/v3/auth/login/socialite/qq?s_url=" +
        redirect +
        "&f_url=" +
        redirect +
        "&action=bind";
    },
    goBindWeixin() {
      this.bindWeixinVisible = true;
    },
    goChangePassword() {
      if (this.user.is_bind_mobile !== 1) {
        this.$message.error("请绑定用户名");
        return;
      }
      this.changePasswordVisible = true;
    },
    getConfig() {
      this.$api.Other.Config().then((res) => {
        this.setConfig(res.data);
      });
    },
    getData() {
      if (this.loading) {
        return;
      }
      this.loading = true;
      this.$api.Member.Detail().then((res) => {
        this.loading = false;
        this.user = res.data;
      });
    },
    uploadAvatar(e) {
      let files = e.target.files;
      if (!files[0].type.match(/.jpg|.png|.jpeg/i)) {
        this.$message.error("图片格式错误,请上传png/jpg/jpeg格式的图片");
        return;
      }
      if (files[0].size > 2048000) {
        this.$message.error("图片大小不超过2M");
        return;
      }
      let formData = new FormData();
      formData.append("file", files[0]);
      axios
        .post("/api/v2/member/detail/avatar", formData)
        .then((res) => {
          if (res.data.code === 0) {
            this.$message.success("上传头像成功");
            this.resetData();
          } else {
            this.$message.error(res.data.message);
          }
        })
        .catch((e) => {
          this.$message.error("上传图片出现错误");
        });
    },
    saveEditNick() {
      if (this.formLoading) {
        return;
      }
      if (!this.form.nick_name) {
        this.$message.error("请输入昵称");
        return;
      }
      this.formLoading = true;
      this.$api.Member.NicknameChange({ nick_name: this.form.nick_name })
        .then((res) => {
          this.formLoading = false;
          this.$message.success("修改成功");
          this.resetData();
        })
        .catch((e) => {
          this.formLoading = false;
          this.$message.error(e.message);
        });
    },
    CodeBind(code) {
      if (this.$utils.getSessionLoginCode(code)) {
        return;
      }
      this.$utils.saveSessionLoginCode(code);
      this.$api.Auth.CodeBind({ code: code })
        .then((res) => {
          this.$message.success("绑定成功");
          this.success();
        })
        .catch((e) => {
          this.$message.error(e.message);
        });
    },
    destroyUser() {
      this.destroyUserVisible = true;
    },
    cancel() {
      this.changePasswordVisible = false;
      this.mobileValidateVisible = false;
      this.bindMobileVisible = false;
      this.bindNewmobileVisible = false;
      this.destroyUserVisible = false;
      this.app = null;
      this.openmask = false;
      this.bindWeixinVisible = false;
    },
    cancelBindQQ() {
      this.app = "qq";
      this.openmask = true;
    },
    cancelBindWeixin() {
      this.app = "wechat";
      this.openmask = true;
    },
    success() {
      this.cancel();
      this.resetData();
      this.getConfig();
    },
    successMobileValidate(sign) {
      this.bindMobileSign = sign;
      this.cancel();
      this.bindMobileVisible = true;
    },
    submitHandle() {
      this.$api.Member.CancelBind(this.app).then((res) => {
        this.$message.success("解绑成功");
        this.success();
      });
    },
  },
};
</script>
<style lang="less" scoped>
.content {
  width: 100%;
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
      overflow: hidden;
      display: flex;
      flex-direction: column;
      box-sizing: border-box;
      padding: 30px 0px;
      animation: scaleBig 0.3s;

      .dialog-tabs {
        width: 100%;
        height: 20px;
        display: flex;
        flex-direction: row;
        position: relative;
        box-sizing: border-box;
        padding: 0px 30px;

        .btn-close {
          width: 16.25px;
          height: 16.25px;
          position: absolute;
          right: 22px;
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
          font-size: 18px;
          font-weight: 500;
          color: #333333;
          line-height: 20px;
        }
      }
      .box {
        width: 100%;
        margin-top: 50px;
        display: flex;
        flex-direction: column;
        box-sizing: border-box;
        padding: 0px 30px;
        margin-bottom: 50px;
        .input-item {
          width: 100%;
          height: 60px;
          font-size: 16px;
          font-weight: 400;
          color: #333333;
          line-height: 30px;
        }
      }
      .btn-box {
        width: 100%;
        height: 74px;
        background: #ffffff;
        box-shadow: 0px -2px 4px 0px rgba(102, 102, 102, 0.05);
        box-sizing: border-box;
        padding: 15px 30px;
        display: flex;
        flex-direction: row-reverse;
        .submit {
          width: 88px;
          height: 44px;
          background: #3ca7fa;
          border-radius: 4px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 14px;
          font-weight: 400;
          color: #ffffff;
          line-height: 14px;
          outline: none;
          &:hover {
            opacity: 0.8;
          }
        }
        .cancel {
          width: 88px;
          height: 44px;
          border: 1px solid #cccccc;
          background: #fff;
          border-radius: 4px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 14px;
          font-weight: 400;
          color: #333333;
          line-height: 14px;
          outline: none;
          margin-right: 30px;
          &:hover {
            opacity: 0.8;
          }
        }
      }
    }
  }
  .box {
    width: 1200px;
    margin: 0 auto;
    display: flex;
    flex-direction: row;
    margin-top: 30px;
    .project-box {
      margin-left: 30px;
      width: 999px;
      height: auto;
      display: flex;
      flex-direction: column;
      .user-box {
        width: 100%;
        height: 140px;
        background: #3ca7fa;
        border-radius: 8px;
        box-sizing: border-box;
        padding: 30px;
        margin-bottom: 20px;
        display: flex;
        flex-direction: row;
        .avatar {
          width: 80px;
          height: 80px;
          overflow: hidden;
          border-radius: 50%;
          margin-right: 20px;
          img {
            width: 80px;
            height: 80px;
            border-radius: 50%;
          }
        }
        .user-info {
          width: 403px;
          display: flex;
          flex-direction: column;
          .user-top {
            width: 403px;
            margin-top: 8px;
            height: 28px;
            display: flex;
            flex-direction: row;
            align-items: center;
            .nickname {
              margin-right: 15px;
              height: 24px;
              font-size: 24px;
              font-weight: 600;
              color: #ffffff;
              line-height: 24px;
            }
            .normal {
              width: 72px;
              height: 28px;
              background: #ffffff;
              border-radius: 4px;
              display: flex;
              align-items: center;
              justify-content: center;
              font-size: 14px;
              line-height: 14px;
              font-weight: 400;
              color: #3ca7fa;
            }
            .role {
              background: #fff566;
              border-radius: 4px;
              font-size: 14px;
              font-weight: 400;
              color: #3ca7fa;
              line-height: 14px;
              padding: 7px 8px;
            }
          }
          .expiration-time {
            width: 403px;
            margin-top: 20px;
            height: 14px;
            font-size: 14px;
            font-weight: 400;
            color: #ffffff;
            line-height: 14px;
          }
        }
        .value-box {
          width: 406px;
          display: flex;
          flex-direction: row;
          justify-content: space-between;
          box-sizing: border-box;
          padding-top: 10px;
          .item {
            display: flex;
            flex-direction: column;
            .value {
              text-align: center;
              height: 30px;
              font-size: 30px;
              font-weight: 600;
              color: #ffffff;
              line-height: 30px;
              margin-bottom: 16px;
            }
            .name {
              text-align: center;
              height: 14px;
              font-size: 14px;
              font-weight: 400;
              color: #ffffff;
              line-height: 14px;
            }
          }
        }
      }
      .user-profile {
        flex: 1;
        position: relative;
        width: 100%;
        background: #ffffff;
        border-radius: 8px;
        box-sizing: border-box;
        padding: 30px 30px 50px 30px;
        .del-user {
          position: absolute;
          top: 30px;
          right: 30px;
          width: auto;
          height: 14px;
          font-size: 14px;
          font-weight: 400;
          color: #cccccc;
          line-height: 14px;
          cursor: pointer;
          z-index: 20;
          &:hover {
            opacity: 0.8;
          }
        }
        .tabs {
          width: 100%;
          height: 34px;
          position: relative;
          box-sizing: border-box;
          display: flex;
          flex-direction: row;
          padding: 0px;
          margin-bottom: 30px;
          .item-tab {
            width: auto;
            height: 14px;
            font-size: 14px;
            font-weight: 400;
            color: #666666;
            line-height: 14px;
            margin-right: 50px;
            cursor: pointer;
            position: relative;
            text-align: center;
            &.active {
              font-weight: 600;
              color: #3ca7fa;
            }
            .actline {
              width: 56px;
              height: 4px;
              background: #3ca7fa;
              position: absolute;
              left: 0px;
              top: 29px;
            }
          }
        }
        .project-content {
          width: 100%;
          display: flex;
          flex-direction: column;
          .btn-box {
            width: 100%;
            height: 40px;
            display: flex;
            align-items: center;
            flex-direction: row;
            .button-submit {
              width: 104px;
              height: 40px;
              margin-left: 90px;
              background: #cccccc;
              border-radius: 4px;
              display: flex;
              align-items: center;
              justify-content: center;
              font-size: 16px;
              font-weight: 400;
              color: #ffffff;
              cursor: pointer;
              &:hover {
                opacity: 0.8;
              }
              &.active {
                background: #3ca7fa;
              }
            }
          }
          .item-line {
            width: 100%;
            display: flex;
            align-items: center;
            flex-direction: row;
            justify-content: space-between;
            &:not(:last-of-type) {
              margin-bottom: 50px;
            }
            .item-left {
              display: flex;
              flex-direction: row;
              align-items: center;
              .item-name {
                width: 110px;
                height: 14px;
                font-size: 14px;
                font-weight: 400;
                color: #666666;
                line-height: 14px;
                margin-right: 101px;
                display: flex;
                flex-direction: row;
                align-items: center;
                img {
                  width: 20px;
                  height: 20px;
                  margin-right: 10px;
                }
              }
              .item-tit {
                width: 56px;
                height: 14px;
                font-size: 14px;
                font-weight: 400;
                color: #666666;
                line-height: 14px;
                margin-right: 34px;
              }
              .item-sptit {
                width: 70px;
                height: 14px;
                font-size: 14px;
                font-weight: 400;
                color: #666666;
                line-height: 14px;
                margin-right: 20px;
              }
              .item-radio {
                display: flex;
                height: 20px;
                flex-direction: row;
                align-items: center;
                .boy {
                  display: flex;
                  height: 20px;
                  flex-direction: row;
                  align-items: center;
                  input {
                    width: 14px;
                    height: 14px;
                    margin-right: 10px;
                  }
                  label {
                    font-size: 14px;
                    font-weight: 400;
                    color: #333333;
                    line-height: 14px;
                    margin-right: 30px;
                  }
                }
                .girl {
                  display: flex;
                  height: 20px;
                  flex-direction: row;
                  align-items: center;
                  input {
                    width: 14px;
                    height: 14px;
                    margin-right: 10px;
                  }
                  label {
                    font-size: 14px;
                    font-weight: 400;
                    color: #333333;
                    line-height: 14px;
                  }
                }
              }
              .item-btn {
                width: auto;
                height: 14px;
                font-size: 14px;
                font-weight: 400;
                color: #3ca7fa;
                line-height: 14px;
                cursor: pointer;
              }
              .item-avatar {
                width: 48px;
                height: 48px;
                border-radius: 24px;
                overflow: hidden;
                position: relative;
                .avatar {
                  width: 48px;
                  height: 48px;
                  border-radius: 24px;
                }
                .input-avatar {
                  position: absolute;
                  top: 0;
                  left: 0;
                  opacity: 0;
                  width: 48px;
                  height: 48px;
                  border-radius: 24px;
                }
              }
              .item-thumb {
                width: 160px;
                height: 90px;
                background: #f4fafe;
                cursor: pointer;
                margin-right: 50px;
                position: relative;
                .icon {
                  position: absolute;
                  width: 30px;
                  height: 30px;
                  top: 30px;
                  left: 65px;
                }
                img {
                  width: 100%;
                  height: 100%;
                  cursor: pointer;
                }
              }
              .item-btn {
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
              .item-value {
                font-size: 14px;
                font-weight: 400;
                color: #333333;
                line-height: 14px;
                .input {
                  width: 224px;
                  height: 36px;
                  background: #ffffff;
                  border-radius: 4px;
                  border: 1px solid #dcdfe6;
                  padding-left: 15px;
                }
              }
            }
            .item-right {
              display: flex;
              flex-direction: row;
              align-items: center;
              .btn {
                cursor: pointer;
                height: 14px;
                font-size: 14px;
                font-weight: 400;
                color: #999999;
                line-height: 14px;
                &.act {
                  color: #3ca7fa;
                }
              }
              .tip {
                margin-left: 10px;
                height: 14px;
                font-size: 14px;
                font-weight: 400;
                color: #999999;
                line-height: 14px;
              }
            }
          }
        }
      }
    }
  }
}
</style>
