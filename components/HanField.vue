<template>
  <div class="han-field" @click="$emit('click')">
    <input
      v-if="!hasArrow"
      class="input"
      :type="inputType"
      :value="value"
      :disabled="disabled"
      :autofocus="autofocus"
      :style="{ color: inputColor }"
      :placeholder="placeholder"
      @input="($event) => $emit('input', $event.value)"
      @change="($event) => $emit('input', $event.value)"
      @focus="showIcon = true"
      @blur="handleBlur"
    />
    <image
      class="icon"
      v-if="value && showIcon"
      :src="closeIcon"
      resize="contain"
      @click="$emit('input', '')"
    />
    <image
      class="icon"
      v-if="type === 'password'"
      :src="inputType === 'password' ? closeEyes : openEyes"
      resize="contain"
      style="margin-left: 10px"
      @click="inputType = inputType === 'password' ? 'text' : 'password'"
    />
    <text
      v-if="hasArrow"
      class="input"
      :style="{ color: value ? inputColor : '#ccc' }"
      >{{ value || placeholder }}</text
    >
    <image v-if="hasArrow" class="icon" :src="arrowIcon" resize="contain" />
    <slot name="code" />
    <slot />
  </div>
</template>

<script>
import commonConf from "@/configs/common.conf";
import tools from "@/utils/tools";

export default {
  props: {
    type: {
      type: String,
      default: "text",
    },
    value: {
      type: String,
      default: "",
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    autofocus: {
      type: Boolean,
      default: false,
    },
    placeholder: {
      type: String,
      default: "",
    },
    hasArrow: {
      type: Boolean,
      default: false,
    },
    inputColor: {
      type: String,
      default: "#333",
    },
    validated: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      closeIcon: commonConf.imgUrl + "img_deleted.png",
      arrowIcon: commonConf.imgUrl + "img_arrow.png",
      openEyes: commonConf.imgUrl + "zyj.png",
      closeEyes: commonConf.imgUrl + "yj.png",
      inputType: "",
      showIcon: false,
    };
  },
  methods: {
    handleBlur() {
      this.showIcon = false;
      !this.value && tools.toast(this.placeholder);
      const hasCardStr = this.placeholder.indexOf("证件号码") !== -1;
      const hasPhoneStr =
        this.placeholder.indexOf("联系方式") !== -1 ||
        this.placeholder.indexOf("手机号码") !== -1;
      const hasEmailStr = this.placeholder.indexOf("邮箱号码") !== -1;
      const hasLoginnameStr = this.placeholder.indexOf("登录名") !== -1;
      if (this.validated && this.value) {
        if (
          (hasCardStr && !tools.validateCardNo(this.value)) ||
          (hasPhoneStr && !tools.validatePhone(this.value)) ||
          (hasEmailStr && !tools.validateEmail(this.value)) ||
          (hasLoginnameStr && !tools.validateName(this.value))
        )
          tools.toast(
            `${this.placeholder.slice(0, 3)}正确的${this.placeholder.slice(3)}`
          );
      }
      this.$emit("blur");
    },
  },
  created() {
    this.inputType = this.type;
  },
};
</script>

<style scoped>
.han-field {
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: #eee;
  flex-direction: row;
  align-items: center;
}
.input {
  flex: 1;
  height: 96px;
  line-height: 96px;
  font-family: PingFangSC-Regular;
  font-size: 34px;
  placeholder-color: #ccc;
}
.icon {
  width: 44px;
  height: 96px;
}
</style>
