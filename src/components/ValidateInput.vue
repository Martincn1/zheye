<template>
  <div class="validate-input-container pb-3">
    <input v-bind="$attrs" class="form-control" :class="{'is-invalid': inputRef.error}" :value="inputRef.val" @blur="validateInput" @input="updateValue">
    <span v-if="inputRef.error" class="invalid-feedback">{{inputRef.message}}</span>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, PropType } from 'vue'
const emailReg = /^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/
const passwordReg = /(?=.*\d)(?=.*[a-zA-Z])(?=.*[^a-zA-Z0-9]).{6,16}/
interface RuleProp {
  type: 'required' | 'email' | 'password';
  message: string;
}
export type RulesProp = RuleProp[]
export default defineComponent({
  props: {
    rules: Array as PropType<RulesProp>,
    modelValue: String // 第一步设置modelValue属性
  },
  inheritAttrs: false, // 不让根元素继承属性
  setup (props, context) {
    console.log(context.attrs)
    const inputRef = reactive({
      val: props.modelValue || '',
      error: false,
      message: ''
    })
    // vue3.0v-model版本
    const updateValue = (e: KeyboardEvent) => {
      const targetValue = (e.target as HTMLInputElement).value
      inputRef.val = targetValue
      // 第二部update:modelValue事件
      context.emit('update:modelValue', targetValue) // 这里的context相当于vue2的this
    }
    const validateInput = () => {
      if (props.rules) {
        const allPassed = props.rules.every((rule) => {
          let passed = true
          inputRef.message = rule.message
          switch (rule.type) {
            case 'required':
              passed = inputRef.val.trim() !== ''
              break
            case 'email':
              passed = emailReg.test(inputRef.val)
              break
            case 'password':
              passed = passwordReg.test(inputRef.val)
              break
            default:
              break
          }
          return passed
        })
        inputRef.error = !allPassed
      }
    }
    return {
      inputRef,
      validateInput,
      updateValue
    }
  }
})
</script>
