<template>
  <div class="han-radio">
    <div
      class="item"
      v-for="(item, index) in data"
      :key="index"
      @click="change(index)"
    >
      <image v-if="!item.isCheck" class="icon" :src="normal" />
      <image v-if="item.isCheck" class="icon" :src="success" />
      <text class="label">{{ item.label }}</text>
    </div>
  </div>
</template>

<script>
import commonConf from "@/configs/common.conf";
import tools from "@/utils/tools";

export default {
  model: {
    prop: "value",
    event: "change",
  },
  props: {
    options: {
      type: Array,
      default: () => [],
    },
    value: {
      type: Number,
    },
  },
  data() {
    return {
      normal: commonConf.imgUrl + "radio_normal.png",
      success: commonConf.imgUrl + "success.png",
      data: [],
    };
  },
  watch: {
    value(val) {
      this.data.forEach((item) => {
        item.isCheck = false;
        item.value === val && (item.isCheck = true);
      });
    },
  },
  methods: {
    change(index) {
      this.data.forEach((item) => (item.isCheck = false));
      this.data[index].isCheck = true;
      this.$emit("change", this.data[index].value);
    },
  },
  created() {
    this.data = this.options.map((item) => ({ ...item, isCheck: false }));
    this.data.forEach((item) => {
      item.value === this.value && (item.isCheck = true);
    });
  },
};
</script>

<style scoped>
.han-radio {
  height: 96px;
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: #eee;
  flex-direction: row;
  align-items: center;
  padding-left: 22px;
}
.item {
  flex-direction: row;
  align-items: center;
  margin-right: 24px;
}
.icon {
  width: 44px;
  height: 44px;
  margin-right: 12px;
}
.label {
  font-family: PingFangSC-Regular;
  font-size: 28px;
  color: #333;
}
</style>
