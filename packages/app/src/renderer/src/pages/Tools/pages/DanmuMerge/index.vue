<!-- 弹幕合并工具 -->
<template>
  <div>
    <div class="center btns" style="margin-bottom: 20px">
      <span v-if="fileList.length !== 0" style="cursor: pointer; color: #958e8e" @click="clear"
        >清空</span
      >
      <n-button @click="addVideo" style="margin-left: 10px"> 添加 </n-button>
      <n-button
        type="primary"
        @click="convert"
        title="立即合并(ctrl+enter)"
        style="margin-left: 10px"
      >
        立即合并
      </n-button>
    </div>
    <FileSelect
      ref="fileSelect"
      v-model="fileList"
      :sub-extensions="supportedVideoExtensions"
      :main-extensions="['xml']"
      area-placeholder="请选择弹幕文件"
    >
      <template #subFileOn>
        <svg
          class="icon"
          viewBox="0 0 1024 1024"
          version="1.1"
          xmlns="http://www.w3.org/2000/svg"
          p-id="900"
          width="20"
          height="20"
        >
          <path
            d="M825.6 892.8 198.4 892.8C124.16 892.8 64 832.64 64 758.4L64 265.6C64 191.36 124.16 131.2 198.4 131.2L377.6 131.2 198.4 310.4l44.8 0 179.2-179.2L601.6 131.2 422.4 310.4l44.8 0 179.2-179.2L825.6 131.2 691.2 310.4l44.8 0 129.408-172.48C920.064 154.944 960 205.312 960 265.6l0 492.736C960 832.64 899.84 892.8 825.6 892.8zM870.4 355.2 153.6 355.2l0 345.28c0 56.768 48.128 102.72 107.52 102.72l501.76 0c59.392 0 107.52-46.016 107.52-102.72L870.4 355.2zM422.4 757.504 422.4 401.152l0.448-0.448L602.24 579.328l-179.456 178.624L422.4 757.504z"
            p-id="901"
          ></path>
        </svg>
      </template>
      <template #subFileOff>
        <svg
          class="icon"
          viewBox="0 0 1024 1024"
          xmlns="http://www.w3.org/2000/svg"
          fill="currentColor"
          width="20"
          height="20"
        >
          <!-- 原始视频图标 -->
          <path
            d="M825.6 892.8H198.4C124.16 892.8 64 832.64 64 758.4V265.6C64 191.36 124.16 131.2 198.4 131.2H377.6L198.4 310.4h44.8l179.2-179.2H601.6L422.4 310.4h44.8l179.2-179.2H825.6L691.2 310.4h44.8l129.408-172.48C920.064 154.944 960 205.312 960 265.6v492.736c0 74.24-60.16 134.4-134.4 134.4zM870.4 355.2H153.6v345.28c0 56.768 48.128 102.72 107.52 102.72h501.76c59.392 0 107.52-46.016 107.52-102.72V355.2zM422.4 757.504V401.152l0.448-0.448 179.392 178.624-179.392 178.176z"
          />

          <!-- 半径为 260 的红色圆圈与斜线 -->
          <circle cx="512" cy="512" r="260" fill="none" stroke="red" stroke-width="45" />
          <line x1="290" y1="290" x2="734" y2="734" stroke="red" stroke-width="45" />
        </svg>
      </template>
    </FileSelect>
  </div>
</template>

<script setup lang="ts">
import hotkeys from "hotkeys-js";
import { supportedVideoExtensions } from "@renderer/utils";

import FileSelect from "@renderer/pages/Tools/pages/Burn/components/FileSelect.vue";

defineOptions({
  name: "DanmuMerge",
});
onActivated(() => {
  hotkeys("ctrl+enter", function () {
    convert();
  });
});
onDeactivated(() => {
  hotkeys.unbind();
});
onUnmounted(() => {
  hotkeys.unbind();
});

const notice = useNotification();

const fileList = ref<{ id: string; title: string; mainFilePath: string; subFilePath?: string }[]>(
  [],
);
const convert = async () => {
  if (fileList.value.length < 2) {
    notice.error({
      title: `至少选择2个弹幕文件`,
      duration: 1000,
    });
    return;
  }
};

const fileSelect = ref<InstanceType<typeof FileSelect> | null>(null);
const addVideo = async () => {
  fileSelect.value?.select();
};

const clear = () => {
  fileList.value = [];
};
</script>

<style scoped lang="less">
.radio-group {
  :deep(.n-radio) {
    align-items: center;
  }
}

.checkmark {
  transform: scale(1.5);
  transform-origin: right bottom;
  background-color: white;
}
</style>
