<template>
  <div class="button">
    <image
      class="code"
      v-if="codeType === '20'"
      :src="codeImage"
      @click="generateCaptchaCode"
    />
    <text class="text" :style="buttonStyle" v-else @click="getCode">{{
      content
    }}</text>
    <div class="line"></div>
  </div>
</template>

<script>
import tools from "@/utils/tools";
import commons from "@/services/commons";
import lightAppJssdk from "@/utils/lightAppJssdk";

export default {
  name: "HanCode",
  // codeType  "00"：个人手机，"01"：个人邮箱，"10"：法人手机，"11"：法人邮箱，"20"：图形验证码
  props: {
    codeType: {
      type: String,
      required: true,
    },
    num: String,
    checked: {
      type: Boolean,
      default: true,
    },
    iscode:String
  },
  data() {
    return {
      codeImage: null,
      content: "获取验证码",
      canClick: true,
      clock: null,
    };
  },
  computed: {
    buttonStyle() {
      if (this.codeType === "00" || this.codeType === "10") {
        if (tools.validatePhone(this.num) && this.canClick && this.checked) {
          return { color: "#1677ff" };
        } else {
          return { color: "#ddd" };
        }
      } else if (this.codeType === "01" || this.codeType === "11") {
        if (tools.validateEmail(this.num) && this.canClick && this.checked) {
          return { color: "#1677ff" };
        } else {
          return { color: "#ddd" };
        }
      }
    },
  },
  methods: {
    /**生成图形验证码 */
    generateCaptchaCode() {
      const captcha = weex.requireModule("captcha");
      captcha.generateCaptchaWithSize(
        {
          width: 170,
          height: 60,
        },
        (res) => {
          if (res.result) {
            if(this.iscode) {
              this.$emit("update",this.iscode);
            let that = this;
            lightAppJssdk.storage.getItem({
                key: "codeImage",
                success: data => {
                    if (data != 'undefined') {
                      that.codeImage =  "data:image/png;base64," + data
                    }
                }
            });
            }else {
              this.$emit("update", res.data.code);
              this.codeImage = "data:image/png;base64," + res.data.image;

              let that = this
              lightAppJssdk.storage.setItem({
                key: "codeImage",
                value: res.data.image
                });

                  }
                }
              }
      );
    },
    /**获取验证码 */
    getCode() {
      if (!this.canClick) return tools.toast("验证码已发送，请稍后再试");
      if (!this.checked) return tools.toast("请输入正确的图形验证码");
      if (this.codeType === "00" || this.codeType === "10") {
        if (!tools.validatePhone(this.num)) return;
        commons
          .sendMobileCode(this.num, this.codeType === "00" ? 1 : 2)
          .then(() => tools.toast("验证码已发送"));
      } else if (this.codeType === "01" || this.codeType === "11") {
        if (!tools.validateEmail(this.num)) return;
        commons
          .sendEmailCode(this.num, this.codeType === "01" ? 1 : 2)
          .then(() => tools.toast("验证码已发送"));
      }
      let time = 10;
      this.canClick = false;
      this.content = time + "s后重新发送";
      this.clock = setInterval(() => {
        time--;
        this.content = time + "s后重新发送";
        if (time < 0) {
          clearInterval(this.clock);
          this.content = "获取验证码";
          this.canClick = true;
        }
      }, 1000);
    },
  },
  created() {
    if(this.codeType == '20') {
      this.generateCaptchaCode();
    }
  },
  beforeDestroy() {
    clearInterval(this.clock);
    this.clock = null;
  },
};
</script>

<style scoped>
.button {
  padding-left: 26px;
  margin-left: 26px;
}
.text {
  line-height: 48px;
  font-family: PingFangSC-Regular;
  font-size: 34px;
}
.line {
  position: absolute;
  left: 0;
  top: 2px;
  bottom: 2px;
  width: 2px;
  background-color: #eee;
}
.code {
  width: 170px;
  height: 60px;
}
</style>
