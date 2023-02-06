<template>
  <scroller :show-scrollbar="false" class="main" v-if="show">
    <div class="content" v-if="flag === '0'">
      <HanField v-model="info.name" placeholder="请输入被授权人姓名" />
      <HanField :value="card" disabled hasArrow @click="selectCard" />
      <HanField
        v-model="info.papersNumber"
        placeholder="请输入被授权人证件号码"
      />
      <HanField v-model="info.mobile" placeholder="请输入被授权人联系方式" />
      <HanField
        v-if="isCheckSms"
        v-model="info.mobileCode"
        placeholder="请输入短信验证码"
      >
        <HanCode slot="code" codeType="10" :num="submitData.mobile" />
      </HanField>
      <HanField
        v-model="info.loginName"
        :disabled="Boolean(iid)"
        placeholder="请输入登录名"
      />
      <HanField
        v-model="info.password"
        type="password"
        placeholder="请输入密码"
      />
      <HanField
        v-model="info.confirmPwd"
        type="password"
        placeholder="请确认密码"
      />
      <HanField
        :value="mode"
        disabled
        hasArrow
        placeholder="请选择授权方式"
        @click="selectMode"
      />
      <HanField
        v-model="info.authStartTime"
        type="date"
        placeholder="请选择授权有效开始时间"
      />
      <HanRadio :options="options" v-model="value" />
      <HanField
        class="input"
        v-model="info.authEndTime"
        type="date"
        placeholder="请选择授权有效结束时间"
      />
    </div>
    <div class="content" v-if="flag === '1'">
      <HanField
        :class="[!rangCheck && !isCheckSms && 'input']"
        :value="frMobileEnc"
        disabled
      />
      <HanField
        :class="[!isCheckSms && 'input']"
        v-if="rangCheck"
        v-model="picCode"
        placeholder="请输入图形验证码"
        @input="picCode === validCode ? (checked = true) : (checked = false)"
      >
        <HanCode
          slot="code"
          codeType="20"
          @update="(val) => (validCode = val)"
        />
      </HanField>
      <HanField
        class="input"
        v-if="isCheckSms"
        v-model="mobileCode"
        placeholder="请输入短信验证码"
      >
        <HanCode
          slot="code"
          codeType="10"
          :num="frMobile"
          :checked="checked || !rangCheck"
        />
      </HanField>
    </div>
    <HanButton title="保存" @click="submit" />
  </scroller>
</template>

<script>
import tools from "@/utils/tools";
import { HanField, HanCode, HanButton, HanRadio } from "@/components";
import lightAppJssdk from "@/utils/lightAppJssdk";
import userCenter from "../../services/userCenter.js";
import moment from "moment";

export default {
  components: {
    HanButton,
    HanField,
    HanCode,
    HanRadio,
  },
  data() {
    return {
      token: null,
      iid: null,
      flag: null,
      cards: [],
      modes: [
        { typeKey: "全权授权", typeValue: 1 },
        { typeKey: "二次授权", typeValue: 2 },
      ],
      options: [
        { label: "一个月", value: 1 },
        { label: "三个月", value: 3 },
        { label: "一年", value: 12 },
        { label: "长期有效", value: -1 },
      ],
      value: 3,
      info: {
        name: "",
        papersType: "111",
        papersNumber: "",
        mobile: "",
        mobileCode: "",
        loginName: "",
        password: "",
        confirmPwd: "",
        authWay: null,
        authStartTime: "",
        authEndTime: "",
      },
      infoNormal: {},
      infoEncrypt: {},
      submitData: {},
      frMobile: null,
      frMobileEnc: "",
      picCode: "",
      validCode: null,
      mobileCode: "",
      isCheckSms: false,
      rangCheck: false,
      show: false,
      checked: false,
    };
  },
  computed: {
    card() {
      return this.cards.find((item) => item.typeValue === this.info.papersType)
        ? this.cards.find((item) => item.typeValue === this.info.papersType)
            .typeKey
        : "";
    },
    mode() {
      return this.modes.find((item) => item.typeValue === this.info.authWay)
        ? this.modes.find((item) => item.typeValue === this.info.authWay)
            .typeKey
        : "";
    },
  },
  watch: {
    info: {
      handler(val) {
        const data = { ...val };
        const {
          name,
          papersNumber,
          mobile,
          loginName,
          authStartTime,
          authEndTime,
        } = data;
        const obj = { name, papersNumber, mobile, loginName };
        Object.keys(obj).forEach((key) => {
          if (Object.hasOwnProperty.call(obj, key)) {
            const element = obj[key];
            if (
              element.indexOf("*") !== -1 &&
              element === this.infoEncrypt[key]
            ) {
              data[key] = this.infoNormal[key];
            } else {
              data[key] = element;
            }
          }
        });
        this.submitData = data;
        if (authEndTime !== "--") {
          this.value = moment(authEndTime).diff(
            moment(authStartTime),
            "months",
            true
          );
        }
      },
      immediate: true,
      deep: true,
    },
    value: {
      handler(val) {
        if (val === -1) {
          this.info.authEndTime = "--";
        } else if (val === 1 || val === 3 || val === 12) {
          this.info.authEndTime = "";
          this.info.authStartTime &&
            (this.info.authEndTime = moment(this.info.authStartTime)
              .add(val, "months")
              .format("YYYY-MM-DD"));
        }
      },
    },
  },
  methods: {
    validateValue(data) {
      if (!data.val) {
        tools.toast(data.tip);
        return false;
      }
      const hasCardStr = data.tip.indexOf("证件号码") !== -1;
      const hasPhoneStr =
        data.tip.indexOf("联系方式") !== -1 ||
        data.tip.indexOf("手机号码") !== -1;
      const hasEmailStr = data.tip.indexOf("邮箱号码") !== -1;
      const hasLoginnameStr = data.tip.indexOf("登录名") !== -1;
      if (
        (hasCardStr && !tools.validateCardNo(data.val)) ||
        (hasPhoneStr && !tools.validatePhone(data.val)) ||
        (hasEmailStr && !tools.validateEmail(data.val)) ||
        (hasLoginnameStr && !tools.validateName(data.val))
      ) {
        tools.toast(`${data.tip.slice(0, 3)}正确的${data.tip.slice(3)}`);
        return false;
      }
      return true;
    },
    selectCard() {
      lightAppJssdk.notification.actionSheet({
        cancelButton: "取消",
        otherButtons: this.cards.map((item) => item.typeKey),
        success: (res) => {
          const index = res.data.buttonIndex;
          this.cards[index] &&
            (this.info.papersType = this.cards[index].typeValue);
        },
      });
    },
    selectMode() {
      lightAppJssdk.notification.actionSheet({
        cancelButton: "取消",
        otherButtons: this.modes.map((item) => item.typeKey),
        success: (res) => {
          const index = res.data.buttonIndex;
          this.modes[index] &&
            (this.info.authWay = this.modes[index].typeValue);
        },
      });
    },
    submit() {
      // 删除
      if (this.flag === "1") {
        const {
          token,
          iid,
          mobileCode,
          picCode,
          validCode,
          rangCheck,
          isCheckSms,
        } = this;
        if (rangCheck) {
          if (!picCode) return tools.toast("请输入图形验证码");
          if (picCode !== validCode)
            return tools.toast("请输入正确的图形验证码");
        }
        if (isCheckSms && !mobileCode) return tools.toast("请输入短信验证码");
        userCenter
          .deleteCorAuthAccount({
            token,
            iid,
            mobileCode,
          })
          .then((res) => {
            res = JSON.parse(res);
            if (res.retcode === "000000") {
              const bc = new BroadcastChannel("isSuccess");
              bc.postMessage("delSuccess");
              lightAppJssdk.router.back();
            }
          });
        return;
      }
      const object = { ...this.submitData };
      for (const key in object) {
        if (Object.hasOwnProperty.call(object, key)) {
          let val = object[key];
          switch (key) {
            case "name":
              object[key] = { val, tip: "请输入被授权人姓名" };
              break;
            case "papersType":
              object[key] = { val, tip: "请选择被授权人证件类型" };
              break;
            case "papersNumber":
              object[key] = { val, tip: "请输入被授权人证件号码" };
              break;
            case "mobile":
              object[key] = { val, tip: "请输入被授权人联系方式" };
              break;
            case "mobileCode":
              if (this.isCheckSms) {
                object[key] = { val, tip: "请输入短信验证码" };
              } else {
                delete object[key];
              }
              break;
            case "loginName":
              object[key] = { val, tip: "请输入登录名" };
              break;
            case "password":
              object[key] = { val, tip: "请输入密码" };
              break;
            case "confirmPwd":
              object[key] = { val, tip: "请确认密码" };
              break;
            case "authWay":
              object[key] = { val, tip: "请选择授权方式" };
              break;
            case "authStartTime":
              object[key] = { val, tip: "请选择授权有效开始时间" };
              break;
            case "authEndTime":
              object[key] = { val, tip: "请选择授权有效结束时间" };
              break;
            default:
              delete object[key];
              break;
          }
        }
      }
      let flag = false;
      for (const key in object) {
        if (Object.hasOwnProperty.call(object, key)) {
          const element = object[key];
          if (!this.validateValue(element)) {
            flag = true;
            break;
          }
        }
      }
      if (flag) return;
      // 新增
      if (!this.iid) {
        const params = { ...this.submitData };
        params.token = this.token;
        userCenter.addCorAuthAccount(params).then((res) => {
          res = JSON.parse(res);
          if (res.retcode === "000000") {
            lightAppJssdk.notification.toast({
              text: res.msg,
              success() {
                const bc = new BroadcastChannel("isSuccess");
                bc.postMessage("success");
                setTimeout(lightAppJssdk.router.back, 1000);
              },
            });
          } else {
            tools.toast(res.msg);
          }
        });
        return;
      }
      // 修改
      const params = { ...this.submitData };
      params.token = this.token;
      params.iid = this.iid;
      params.password === "******" && (params.password = undefined);
      userCenter.modifyCorAuthAccount(params).then((res) => {
        res = JSON.parse(res);
        if (res.retcode === "000000") {
          lightAppJssdk.notification.toast({
            text: res.msg,
            success() {
              const bc = new BroadcastChannel("isSuccess");
              bc.postMessage("success");
              setTimeout(lightAppJssdk.router.back, 1000);
            },
          });
        } else {
          tools.toast(res.msg);
        }
      });
    },
  },
  created() {
    this.flag = tools.getUrlParam("flag");
    lightAppJssdk.storage.getItem({
      key: "frSettings",
      success: (data) => {
        this.rangCheck = JSON.parse(data).rangCheck === "Y";
        this.isCheckSms = JSON.parse(data).isCheckSms === "Y";
        /* this.rangCheck = true;
        this.isCheckSms = true; */
      },
    });
    lightAppJssdk.storage.getItem({
      key: "cerTypeArr",
      success: (data) => {
        this.cards = JSON.parse(data);
      },
    });
    lightAppJssdk.storage.getItem({
      key: "frUserInfo",
      success: (data) => {
        this.frMobile = JSON.parse(data).mobile;
        this.frMobileEnc = tools.noPassByMobile(this.frMobile);
      },
    });
    lightAppJssdk.storage.getItem({
      key: "userToken",
      success: (data) => {
        this.token = JSON.parse(data).token;
        if (tools.getUrlParam("iid") !== "undefined")
          this.iid = tools.getUrlParam("iid");
        if (this.flag === "0" && this.iid) {
          userCenter
            .findCorAuthAccountInfo({
              token: this.token,
              iid: this.iid,
            })
            .then((res) => {
              res = JSON.parse(res);
              if (res.retcode === "000000") {
                const data = JSON.parse(res.data).infoMap;
                data.password = "******";
                data.confirmPwd = "******";
                data.mobileCode = "";
                this.info = { ...data };
                this.infoNormal = { ...data };
                this.info.name = tools.noPassByName(data.name);
                this.info.papersNumber = tools.noPassByCard(data.papersNumber);
                this.info.mobile = tools.noPassByMobile(data.mobile);
                this.info.loginName = tools.noPassByAccount(data.loginName);
                this.infoEncrypt = { ...this.info };
                this.show = true;
              } else {
                tools.toast(res.msg);
              }
            });
          return;
        }
        this.show = true;
      },
    });
  },
};
</script>

<style scoped>
.main {
  padding: 0 24px;
  background-color: #fff;
}
.content {
  /* box-shadow: 0 2px 8px 0 rgba(153, 153, 153, 0.2); */
  border-width: 1px;
  border-style: solid;
  border-color: #eee;
  border-radius: 16px;
  padding: 0 24px;
  margin-top: 32px;
  margin-bottom: 60px;
}
.input {
  border-bottom-width: 0;
}
</style>
