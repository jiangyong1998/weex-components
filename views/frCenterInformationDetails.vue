<template>
  <div class="main" v-if="show">
    <div class="content" v-if="id === '0'">
      <HanField :value="cor" hasArrow disabled @click="selectType" />
      <HanField v-model="frInfo.pgName" placeholder="请输入企业/法人名称" />
      <HanField v-model="frInfo.corNumber" disabled inputColor="#999" />
      <HanField v-model="frInfo.realName" placeholder="请输入法定代表人姓名" />
      <HanField :value="card" hasArrow disabled @click="selectCard" />
      <HanField
        class="input"
        v-model="frInfo.cardNumber"
        placeholder="请输入法定代表人证件号码"
      />
    </div>
    <div class="content" v-if="id === '1'">
      <HanField v-model="sqrInfo.name" placeholder="请输入申请人姓名" />
      <HanField :value="card" hasArrow disabled @click="selectCard" />
      <HanField
        :class="[!rangCheck && 'input']"
        v-model="sqrInfo.papersNumber"
        placeholder="请输入申请人证件号码"
      />
      <HanField
        class="input"
        v-if="rangCheck"
        v-model="picCode"
        placeholder="请输入图形验证码"
      >
        <HanCode
          ref="code"
          slot="code"
          codeType="20"
          @update="(val) => (validCode = val)"
        />
      </HanField>
    </div>
    <HanButton
      :title="id === '0' ? '更新信息' : '修改申请人信息'"
      @click="submit"
    />
  </div>
</template>

<script>
import tools from "@/utils/tools";
import { HanField, HanCode, HanButton } from "@/components";
import lightAppJssdk from "@/utils/lightAppJssdk";
import userCenter from "../../services/userCenter.js";

let humanReviewReg;

export default {
  components: {
    HanButton,
    HanField,
    HanCode,
  },
  data() {
    return {
      id: null,
      token: null,
      frUserInfo: null,
      cards: [],
      types: [],
      frInfo: {
        corType: "",
        pgName: "",
        corNumber: "",
        realName: "",
        cardType: "",
        cardNumber: "",
      },
      sqrInfo: {
        name: "",
        papersType: "",
        papersNumber: "",
      },
      picCode: "",
      validCode: null,
      rangCheck: false,
      show: false,
    };
  },
  computed: {
    card() {
      return this.id === "0"
        ? this.cards.find((item) => item.typeValue === this.frInfo.cardType) &&
            this.cards.find((item) => item.typeValue === this.frInfo.cardType)
              .typeKey
        : this.cards.find(
            (item) => item.typeValue === this.sqrInfo.papersType
          ) &&
            this.cards.find(
              (item) => item.typeValue === this.sqrInfo.papersType
            ).typeKey;
    },
    cor() {
      return (
        this.types.find((item) => item.typeValue === this.frInfo.corType) &&
        this.types.find((item) => item.typeValue === this.frInfo.corType)
          .typeKey
      );
    },
  },
  methods: {
    selectCard() {
      lightAppJssdk.notification.actionSheet({
        cancelButton: "取消",
        otherButtons: this.cards.map((item) => item.typeKey),
        success: (res) => {
          const index = res.data.buttonIndex;
          this.id === "0"
            ? this.cards[index] &&
              (this.frInfo.cardType = this.cards[index].typeValue)
            : this.cards[index] &&
              (this.sqrInfo.papersType = this.cards[index].typeValue);
        },
      });
    },
    selectType() {
      lightAppJssdk.notification.actionSheet({
        cancelButton: "取消",
        otherButtons: this.types.map((item) => item.typeKey),
        success: (res) => {
          const index = res.data.buttonIndex;
          this.types[index] &&
            (this.frInfo.corType = this.types[index].typeValue);
        },
      });
    },
    submit() {
      if (this.id === "0") {
        if (!this.frInfo.pgName) return tools.toast("请输入企业/法人名称");
        if (!this.frInfo.realName) return tools.toast("请输入法定代表人姓名");
        if (!tools.validateCardNo(this.frInfo.cardNumber))
          return tools.toast("请输入正确的法定代表人证件号码");
        const params = { ...this.frInfo };
        params.token = this.token;
        userCenter.modifyCorporationInfo(params).then((res) => {
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
            if (!res.data) {
              tools.toast(res.msg);
              return;
            }
            const isCheck = JSON.parse(res.data).corporationCheck;
            if (!isCheck) {
              humanReviewReg
                ? lightAppJssdk.notification.confirm({
                    title: "认证失败",
                    message:
                      '请仔细核实您所填写的信息是否正确，如确认无误，仍无法认证通过，请选择"人工实名核验"方式，完成申请人信息变更。',
                    buttonLabels: ["取消", "人工实名核验"],
                    success: (res) => {
                      const index = res.data.buttonIndex;
                      if (index === 1)
                        tools.urlBridgeGo(
                          "views/newFrComplain/frComplain",
                          "人工实名核验",
                          {
                            frParams: JSON.stringify({
                              ...this.frUserInfo,
                              token: this.token,
                              businessType: "CHANGE",
                              changeType: 1,
                            }),
                          }
                        );
                    },
                  })
                : lightAppJssdk.notification.confirm({
                    title: "认证失败",
                    message:
                      "请仔细核实您所填写的统一社会信用代码是否正确，如确认无误，仍无法认证通过，请联系*******「人工客服咨询」",
                    buttonLabels: ["确定"],
                  });
            }
          }
        });
        return;
      }
      if (!this.sqrInfo.name) return tools.toast("请输入申请人姓名");
      if (!tools.validateCardNo(this.sqrInfo.papersNumber))
        return tools.toast("请输入正确的申请人证件号码");
      if (this.rangCheck) {
        if (!this.picCode) return tools.toast("请输入图形验证码");
        if (this.picCode !== this.validCode) {
          // this.$refs.code.generateCaptchaCode();
          return tools.toast("请输入正确的图形验证码");
        }
      }
      const params = { ...this.sqrInfo };
      params.token = this.token;
      userCenter.modifyApplicantInfo(params).then((res) => {
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
          if (!res.data) {
            tools.toast(res.msg);
            return;
          }
          const isCheck = JSON.parse(res.data).realNameCheck;
          if (!isCheck) {
            humanReviewReg
              ? lightAppJssdk.notification.confirm({
                  title: "认证失败",
                  message:
                    '请仔细核实您所填写的信息是否正确，如确认无误，仍无法认证通过，请选择"人工实名核验"方式，完成申请人信息变更。',
                  buttonLabels: ["取消", "人工实名核验"],
                  success: (res) => {
                    const index = res.data.buttonIndex;
                    if (index === 1)
                      tools.urlBridgeGo(
                        "views/newFrComplain/frComplain",
                        "人工实名核验",
                        {
                          frParams: JSON.stringify({
                            ...this.frUserInfo,
                            token: this.token,
                            businessType: "CHANGE",
                            changeType: 2,
                          }),
                        }
                      );
                  },
                })
              : lightAppJssdk.notification.confirm({
                  title: "认证失败",
                  message:
                    "请仔细核实您所填写的信息是否正确，如确认无误，仍无法认证通过，请联系*******「人工客服咨询」",
                  buttonLabels: ["确定"],
                });
          }
        }
      });
    },
  },
  created() {
    this.id = tools.getUrlParam("id");
    lightAppJssdk.storage.getItem({
      key: "frSettings",
      success: (data) => {
        humanReviewReg = JSON.parse(data).humanReviewReg === "Y";
        this.rangCheck = JSON.parse(data).rangCheck === "Y";
        // this.rangCheck = true;
      },
    });
    lightAppJssdk.storage.getItem({
      key: "cerTypeArr",
      success: (data) => {
        this.cards = JSON.parse(data);
      },
    });
    lightAppJssdk.storage.getItem({
      key: "paperstypeArr",
      success: (data) => {
        this.types = JSON.parse(data);
      },
    });
    lightAppJssdk.storage.getItem({
      key: "userToken",
      success: (data) => {
        this.token = JSON.parse(data).token;
      },
    });
    lightAppJssdk.storage.getItem({
      key: "frUserInfo",
      success: (data) => {
        const {
          corusercardid: cardNumber,
          cortype: corType,
          corname: pgName,
          corusername: realName,
          corusercardtype: cardType,
          cornumber: corNumber,
          name,
          papersnumber: papersNumber,
          paperstype: papersType,
          mobile,
        } = JSON.parse(data);
        this.frUserInfo = JSON.parse(data);
        this.frInfo = {
          corType,
          pgName,
          corNumber,
          realName,
          cardType,
          cardNumber,
        };
        this.sqrInfo = { name, papersType, papersNumber, mobile };
        this.show = true;
      },
    });
  },
};
</script>

<style scoped>
.main {
  padding: 32px 24px;
  background-color: #fff;
}
.content {
  /* box-shadow: 0 2px 8px 0 rgba(153, 153, 153, 0.2); */
  border-width: 1px;
  border-style: solid;
  border-color: #eee;
  border-radius: 16px;
  padding: 0 24px;
  margin-bottom: 60px;
}
.input {
  border-bottom-width: 0;
}
</style>
