<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  modelValue: string | number
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string | number): void
}>()

const value = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  }
})
</script>

<template>
  <div>
    <input v-model="value" />
    <button @click="() => emit('update:modelValue', 'foo')">Update</button>
  </div>
</template>

<style lang="sass" scoped></style>
