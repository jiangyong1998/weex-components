<template>
  <div class="main" v-if="show">
    <div class="content">
      <div class="title-box">
        <text class="title">法人信息</text>
        <HanButton
          class="button"
          title="修改"
          titleFontSize="24px"
          @click="modify('0')"
        />
      </div>
      <HanField :value="cor" disabled />
      <HanField :value="frUserInfo.corname" disabled />
      <HanField :value="frUserInfo.cornumber" disabled />
      <HanField :value="frUserInfo.corusername" disabled />
      <HanField :value="card" disabled />
      <HanField class="input" :value="frUserInfo.corusercardid" disabled />
    </div>
    <div class="content">
      <div class="title-box">
        <text class="title">申请人信息</text>
        <HanButton
          class="button"
          title="修改"
          titleFontSize="24px"
          @click="modify('1')"
        />
      </div>
      <HanField :value="frUserInfo.name" disabled />
      <HanField class="input" :value="frUserInfo.papersnumber" disabled />
    </div>
  </div>
</template>

<script>
import { HanButton, HanField } from "@/components";
import tools from "@/utils/tools";
import lightAppJssdk from "@/utils/lightAppJssdk";
import commons from "@/services/commons";

export default {
  components: {
    HanButton,
    HanField,
  },
  data() {
    return {
      token: null,
      cards: [],
      types: [],
      frUserInfo: {},
      show: false,
    };
  },
  computed: {
    card() {
      return (
        this.cards.find(
          (item) => item.typeValue === this.frUserInfo.corusercardtype
        ) &&
        this.cards.find(
          (item) => item.typeValue === this.frUserInfo.corusercardtype
        ).typeKey
      );
    },
    cor() {
      return (
        this.types.find((item) => item.typeValue === this.frUserInfo.cortype) &&
        this.types.find((item) => item.typeValue === this.frUserInfo.cortype)
          .typeKey
      );
    },
  },
  methods: {
    modify(id) {
      const title = id === "0" ? "修改法人信息" : "修改账号信息";
      tools.urlBridgeGo("views/newFrCenter/frCenterInformationDetails", title, {
        id,
      });
    },
    getFrUserInfo() {
      commons
        .findcoruserbytoken({ data: { token: this.token } })
        .then((res) => {
          if (res.retcode === "000000") {
            this.frUserInfo = JSON.parse(res.data);
            this.show = true;
            lightAppJssdk.storage.setItem({
              key: "frUserInfo",
              value: JSON.stringify(this.frUserInfo),
            });
          } else {
            tools.toast(res.msg);
          }
        });
    },
    init() {
      lightAppJssdk.storage.getItem({
        key: "userToken",
        success: (data) => {
          this.token = JSON.parse(data).token;
          this.getFrUserInfo();
        },
      });
    },
  },
  created() {
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
    this.init();
    const bc = new BroadcastChannel("isSuccess");
    bc.onmessage = () => {
      this.init();
    };
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
  margin-bottom: 24px;
}
.title-box {
  height: 96px;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: #eee;
}
.title {
  font-family: PingFangSC-Semibold;
  font-weight: 600;
  font-size: 34px;
  color: #333333;
}
.button {
  width: 120px;
  height: 49px;
  border-radius: 24.5px;
}
.input {
  border-bottom-width: 0;
}
</style>
