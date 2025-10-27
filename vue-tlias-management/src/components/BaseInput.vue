<template>
  <el-form-item :label="label" :prop="fieldProp" :rules="rules">
    <el-input
        v-model="inputValue"
        :placeholder="placeholder"
        :disabled="disabled"
        @input="handleInput"
      ></el-input>
  </el-form-item>
</template>

<script setup>
import { defineProps, defineEmits, ref, watch } from 'vue'

const props = defineProps({
  label: { type: String, required: true },
  placeholder: { type: String, default: '' },
  fieldProp: { type: String, required: true },
  disabled: { type: Boolean, default: false },
  modelValue: { type: String, default: '' },
  rules: { type: Array, default: () => [] }
})

const emit = defineEmits(['update:modelValue'])
const inputValue = ref(props.modelValue)

// 监听prop变化更新本地值
watch(() => props.modelValue, (newVal) => {
  inputValue.value = newVal
})

const handleInput = (value) => {
  emit('update:modelValue', value)
}
</script>