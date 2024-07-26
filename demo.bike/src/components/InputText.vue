<script setup>
const modelValue = defineModel('modelValue');
</script>
<template>
  <input
    type="text"
    class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
    v-model="modelValue"
  />
</template>
<style scoped></style>
