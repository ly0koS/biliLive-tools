<template>
  <PartArea
    v-if="fileList.length !== 0"
    v-model="fileList"
    :sort="props.sort"
    :placeholder="props.inputPlaceholder"
    @add-subFile="addSubFile"
  >
    <template #subFileOn> <slot name="subFileOn"></slot> </template>
    <template #subFileOff>
      <slot name="subFileOff"></slot>
    </template>
  </PartArea>
  <FileArea
    v-else
    :extensions="extensions"
    :desc="props.areaPlaceholder"
    @change="addOldFile"
  ></FileArea>
</template>

<script setup lang="ts">
import FileArea from "@renderer/components/FileArea.vue";
import PartArea from "./PartArea.vue";
import showDirectoryDialog from "@renderer/components/showDirectoryDialog";

import { supportedVideoExtensions, uuid } from "@renderer/utils";

const fileList = defineModel<
  {
    id: string;
    title: string;
    mainFilePath: string;
    subFilePath?: string;
  }[]
>({ required: true });

const props = withDefaults(
  defineProps<{
    sort?: boolean;
    inputPlaceholder?: string;
    areaPlaceholder?: string;
    mainExtensions?: string[];
    subExtensions?: string[];
  }>(),
  {
    sort: true,
    inputPlaceholder: "请输入",
    areaPlaceholder: "请选择视频文件",
    mainExtensions: () => supportedVideoExtensions,
    subExtensions: () => [],
  },
);

const extensions = computed(() => {
  return [...props.mainExtensions, ...props.subExtensions];
});

const addOldFile = (data: { name: string; path: string }[]) => {
  handleFiles(data.map((item) => item.path));
};

const select = async () => {
  let files: string[] | undefined = [];
  if (window.isWeb) {
    files = await showDirectoryDialog({
      type: "file",
      multi: true,
      exts: extensions.value,
    });
  } else {
    files = await window.api.openFile({
      multi: true,
      filters: [
        {
          name: "file",
          extensions: extensions.value,
        },
        {
          name: "所有文件",
          extensions: ["*"],
        },
      ],
    });
  }

  if (!files) return;
  if (files.length === 0) return;

  handleFiles(files);
};

const handleFiles = (files: string[]) => {
  const danmuFiles: string[] = [];
  const videoFiles: string[] = [];
  files.forEach((file) => {
    if (props.subExtensions.some((ext) => file.endsWith(ext))) {
      danmuFiles.push(file);
    } else {
      videoFiles.push(file);
    }
  });

  const newFiles = videoFiles
    .filter((file) => {
      return !fileList.value.some((item) => item.mainFilePath === file);
    })
    .map((file) => ({
      id: uuid(),
      title: window.path.parse(file).name,
      mainFilePath: file,
      subFilePath: "",
    }));

  const allFiles = fileList.value.concat(newFiles);
  const danmuItems = danmuFiles.map((file) => {
    return {
      path: file,
      name: window.path.parse(file).name,
    };
  });

  if (danmuItems) {
    allFiles.forEach((item) => {
      if (item.subFilePath) return;
      const videoName = window.path.parse(item.mainFilePath).name;

      const danmuItem = danmuItems.find((item) => item.name === videoName);
      if (danmuItem) {
        item.subFilePath = danmuItem.path;
      }
    });
  }

  fileList.value = allFiles;
};

const addSubFile = async (index: number) => {
  let files: string[] | undefined = [];
  if (window.isWeb) {
    files = await showDirectoryDialog({
      type: "file",
      multi: false,
      exts: props.subExtensions,
    });
  } else {
    files = await window.api.openFile({
      multi: false,
      filters: [
        {
          name: "file",
          extensions: props.subExtensions,
        },
        {
          name: "所有文件",
          extensions: ["*"],
        },
      ],
    });
  }
  console.log("files", files);

  if (!files) return;
  if (files.length === 0) return;

  fileList.value[index].subFilePath = files[0];
};

defineExpose({
  select,
});
</script>

<style scoped></style>
