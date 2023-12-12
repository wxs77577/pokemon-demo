<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps({
  modelValue: { type: Number, required: true },
  total: { type: Number, required: true },
  maxCount: { type: Number, default: 10 },
  limit: { type: Number, required: true },
  loading: Boolean
})
const emit = defineEmits(['update:modelValue'])
const pageCount = computed(() => {
  return Math.floor(props.total / props.limit) + 1
})
const pages = computed(() => {
  const numbers = [props.modelValue]
  let offset = 1
  let left = 0, right = 0
  while (numbers.length < props.maxCount) {
    left = props.modelValue - offset
    right = props.modelValue + offset
    if (left >= 1) {
      numbers.unshift(left)
    }
    if (right <= pageCount.value) {
      numbers.push(right)
    }
    offset++
    if (offset > props.maxCount) break;
  }
  if (left > 2) {
    numbers.unshift(0)
  }
  if (left > 1) {
    numbers.unshift(1)
  }
  
  if (right < pageCount.value - 1) {
    numbers.push(0)
  }
  if (right < pageCount.value) {
    numbers.push(pageCount.value)
  }
  return numbers
})
</script>
<template>
  <div class="join flex justify-center flex-wrap" v-show="total > 0">
    <button :disabled="loading || i === 0" class="join-item btn btn-square" :class="{ 'btn-primary': modelValue === i }"
      v-for="i in pages" @click="emit('update:modelValue', i)"
      >{{ i === 0 ? '...' : i }}</button>
  </div>
</template>