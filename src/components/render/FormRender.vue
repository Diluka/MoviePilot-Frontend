<script lang="ts" setup>
import { type PropType, ref } from 'vue'

// 组件接口
interface RenderProps {
  component: string
  text: string
  content?: any
  props?: any
}

// 输入参数
const elementProps = defineProps({
  config: Object as PropType<RenderProps>,
  form: Object as PropType<any>,
})

// 配置元素
const formItem = ref<RenderProps>(elementProps.config || {
  component: 'div',
  text: '',
  props: {},
  content: [],
})

// 配置数据
const formData = ref<any>(elementProps.form || {})
</script>

<template>
  <Component
    :is="formItem.component"
    v-bind="formItem.props"
    v-model="formData[formItem.props?.model || '']"
  >
    {{ formItem.text }}
    <FormRender
      v-for="(innerItem, innerIndex) in (formItem.content || [])"
      :key="innerIndex"
      v-model="formData[innerItem.props?.model || '']"
      :config="innerItem"
      :form="formData"
    />
  </Component>
</template>
