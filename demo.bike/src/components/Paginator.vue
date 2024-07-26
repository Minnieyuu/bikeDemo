<script setup>
import { computed, ref, watch } from 'vue';

const props = defineProps(['totalRecords', 'rows', 'isReset']);
const emits = defineEmits(['page']);

const rows = computed(() => (props.rows ? props.rows : 10));
const totalPages = computed(() => Math.ceil(props.totalRecords / rows.value));
const isReset = computed(() => props.isReset);

// 當前頁數
const currentPage = ref(1);
// 顯示頁數
const maxPage = 10;

// 計算當前顯示頁碼最多十頁
const currentPageList = computed(() => {
  let result = [];
  // 取得當前頁碼的前五頁
  const start = currentPage.value - 5 > 0 ? currentPage.value - 5 : 1;

  // 取得當前頁碼的後五頁
  const end = start + maxPage - 1 <= totalPages.value ? start + maxPage - 1 : totalPages.value;
  for (let i = start; i <= end; i++) {
    result.push(i);
  }
  return result;
});

// 切換頁數
const changePage = (page) => {
  currentPage.value = page;
};

// 前一頁
const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value -= 1;
  }
};

// 下一頁
const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value += 1;
  }
};

watch(currentPage, (val) => {
  emits('page', val);
});

watch(isReset, (val) => {
  if (val) {
    currentPage.value = 1;
  }
});
</script>
<template>
  <nav aria-label="Page navigation example" class="text-center mt-3">
    <ul class="inline-flex -space-x-px text-base h-10">
      <li>
        <a
          href="#"
          class="flex items-center justify-center px-4 h-10 ms-0 leading-tight text-gray-500 bg-white border border-e-0 border-gray-300 rounded-s-lg hover:bg-gray-100 hover:text-gray-700"
          @click="prevPage"
          :class="[currentPage === 1 ? 'cursor-not-allowed' : '']"
          >上一頁</a
        >
      </li>
      <li v-for="(item, index) in currentPageList" :key="item">
        <a
          :class="[
            currentPage === item
              ? 'bg-blue-600 text-white hover:bg-blue-700'
              : 'text-gray-500 bg-white hover:bg-gray-100 '
          ]"
          href="#"
          class="flex items-center justify-center px-4 h-10 leading-tight border border-gray-300"
          @click.prevent="changePage(item)"
          >{{ item }}</a
        >
      </li>
      <li>
        <a
          href="#"
          class="flex items-center justify-center px-4 h-10 leading-tight text-gray-500 bg-white border border-gray-300 rounded-e-lg hover:bg-gray-100 hover:text-gray-700"
          @click.prevent="nextPage"
          :class="[currentPage === totalPages ? 'cursor-not-allowed' : '']"
          >下一頁</a
        >
      </li>
    </ul>
  </nav>
</template>
<style scoped></style>
