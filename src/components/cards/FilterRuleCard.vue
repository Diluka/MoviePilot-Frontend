<script lang="ts" setup>
// 输入参数
const props = defineProps({
  pri: String,
  rules: Array as PropType<string[]>,
  width: String,
  height: String,
})

// 定义触发的自定义事件
const emit = defineEmits(['close', 'changed'])

// 按钮点击
function onClose() {
  emit('close')
}

// 选项变化
function filtersChanged(value: string[]) {
  emit('changed', props.pri, value)
}

// 过滤规则下拉框
const selectFilterOptions = ref<{ [key: string]: string }[]>([
  { title: '特效字幕', value: ' SPECSUB ' },
  { title: '中文字幕', value: ' CNSUB ' },
  { title: '分辨率: 4K', value: ' 4K ' },
  { title: '分辨率: 1080P', value: ' 1080P ' },
  { title: '分辨率: 720P', value: ' 720P ' },
  { title: '排除: 720P', value: ' !720P ' },
  { title: '质量: 蓝光原盘', value: ' BLU ' },
  { title: '排除: 蓝光原盘', value: ' !BLU ' },
  { title: '质量: BLURAY', value: ' BLURAY ' },
  { title: '排除: BLURAY', value: ' !BLURAY ' },
  { title: '质量: UHD', value: ' UHD ' },
  { title: '排除: UHD', value: ' !UHD ' },
  { title: '质量: REMUX', value: ' REMUX ' },
  { title: '排除: REMUX', value: ' !REMUX ' },
  { title: '质量: WEB-DL', value: ' WEBDL ' },
  { title: '排除: WEB-DL', value: ' !WEBDL ' },
  { title: '编码: H265', value: ' H265 ' },
  { title: '排除: H265', value: ' !H265 ' },
  { title: '编码: H264', value: ' H264 ' },
  { title: '排除: H264', value: ' !H264 ' },
  { title: '效果: 杜比视界', value: ' DOLBY ' },
  { title: '排除: 杜比视界', value: ' !DOLBY ' },
  { title: '效果: HDR', value: ' HDR ' },
  { title: '排除: HDR', value: ' !HDR ' },
  { title: '国语配音', value: ' CNVOI ' },
  { title: '排除: 国语配音', value: ' !CNVOI ' },
  { title: '促销: 免费', value: ' FREE ' },
])
</script>

<template>
  <VCard variant="tonal" :width="props.width" :height="props.height">
    <DialogCloseBtn @click="onClose" />
    <VCardItem>
      <VCardTitle>优先级 {{ props.pri }}</VCardTitle>
      <VRow>
        <VCol>
          <VSelect
            :key="props.pri"
            v-model="props.rules"
            variant="underlined"
            :items="selectFilterOptions"
            chips
            label=""
            multiple
            @update:modelValue="filtersChanged"
          />
        </VCol>
      </VRow>
    </VCardItem>
  </VCard>
</template>
