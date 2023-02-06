<template>
  <div class="main" v-if="show">
    <div class="content">
      <div class="title-box">
        <text class="title">授权账号</text>
        <han-button
          class="button"
          title="+ 添加"
          titleFontSize="24px"
          @click="jump('新增账号信息')"
        />
      </div>
      <scroller :show-scrollbar="false" style="flex: 1">
        <div class="info-box" v-for="item in accountList" :key="item.iid">
          <div class="info-left">
            <text class="info">三级法人账号：{{ item.loginName }}</text>
            <text class="info">被授权人姓名：{{ item.name }}</text>
            <text class="info">授权截止时间：{{ item.authEndTime }}</text>
          </div>
          <div class="info-right">
            <image
              :src="edtiorImg"
              class="img"
              @click="jump('修改账号信息', item.iid)"
            />
            <image :src="deleteImg" class="img" @click="del(item.iid)" />
          </div>
        </div>
      </scroller>
    </div>
  </div>
</template>

<script>
import commonConf from "@/configs/common.conf";
import { HanButton } from "@/components";
import tools from "@/utils/tools";
import lightAppJssdk from "@/utils/lightAppJssdk";
import userCenter from "../../services/userCenter.js";

export default {
  components: { HanButton },
  data() {
    return {
      token: null,
      edtiorImg: commonConf.imgUrl + "editor-img.png",
      deleteImg: commonConf.imgUrl + "delete-img.png",
      accountList: [],
      show: false,
    };
  },
  methods: {
    jump(title, iid) {
      tools.urlBridgeGo("views/newFrCenter/frAddDelInformation", title, {
        iid,
        flag: "0",
      });
    },
    del(iid) {
      lightAppJssdk.notification.confirm({
        title: "删除确认",
        message:
          "该授权账号信息删除后将无法恢复，账号关联记录将无法查询，请谨慎操作！",
        buttonLabels: ["取消", "确定"],
        success(res) {
          const index = res.data.buttonIndex;
          if (index === 1)
            tools.urlBridgeGo(
              "views/newFrCenter/frAddDelInformation",
              "授权人身份验证",
              {
                iid,
                flag: "1",
              }
            );
        },
      });
    },
    findCorAuthAccountList() {
      userCenter
        .findCorAuthAccountList({
          token: this.token,
        })
        .then((res) => {
          res = JSON.parse(res);
          if (res.retcode === "000000") {
            this.accountList = JSON.parse(res.data).accountList;
            this.show = true;
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
          this.findCorAuthAccountList();
        },
      });
    },
  },
  created() {
    this.init();
    const bc = new BroadcastChannel("isSuccess");
    bc.onmessage = (event) => {
      if (event.data === "delSuccess")
        setTimeout(() => {
          tools.toast("删除成功");
        }, 1000);
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
  flex: 1;
}
.title-box {
  height: 96px;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
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
.info-box {
  height: 241px;
  padding: 26px 0;
  flex-direction: row;
  justify-content: space-between;
  border-top-width: 1px;
  border-top-style: solid;
  border-top-color: #eee;
}
.info-left {
  justify-content: space-between;
}
.info {
  font-family: PingFangSC-Regular;
  font-weight: 400;
  font-size: 32px;
  color: #333333;
}
.info-right {
  justify-content: space-around;
}
.img {
  width: 41px;
  height: 41px;
}
</style>
