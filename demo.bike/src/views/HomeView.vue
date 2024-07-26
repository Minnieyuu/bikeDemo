<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import { initFlowbite } from 'flowbite';
import Paginator from '@/components/Paginator.vue';
import InputText from '@/components/InputText.vue';

// 呼叫api https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json
// 取得資料後，將資料存入bikeinfo
/*
"sno"：站點編號，這裡是 500101001。
"sna"：站點名稱，中文為 YouBike2.0_捷運科技大樓站。
"sarea"：站點所在區域，這裡位於大安區。
"mday"：資料更新時間，是在 2024 年 5 月 11 日晚上 20:15:19 更新的。
"ar"：站點地址，這裡是在復興南路二段235號前。
"sareaen"：站點所在區域的英文名稱，是 Daan Dist.。
"snaen"：站點名稱的英文翻譯，是 YouBike2.0_MRT Technology Bldg. Sta.。
"aren"：站點地址的英文翻譯，是 No.235， Sec. 2， Fuxing S. Rd.。
"act"：站點是否在使用中，這裡是 1 代表在使用中。
"srcUpdateTime"：原始資料來源的更新時間，是在 2024 年 5 月 11 日晚上 20:15:26 更新的。
"updateTime"：資料更新時間，是在 2024 年 5 月 11 日晚上 20:15:52。
"infoTime"：資訊時間，是在 2024 年 5 月 11 日晚上 20:15:19。
"infoDate"：資訊日期，是在 2024 年 5 月 11 日。
"total"：總車位數量，這裡是 28 個。
"available_rent_bikes"：可租借的腳踏車數量，這裡是 1 輛。
"latitude"：站點的緯度坐標，這裡是 25.02605。
"longitude"：站點的經度坐標，這裡是 121.5436。
"available_return_bikes"：可歸還的腳踏車數量，這裡是 27 輛。
*/

// YouBike資料
const bikeinfo = ref([]);

// 取得YouBike資料
const getBikeInfo = async () => {
  const response = await fetch(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  const data = await response.json();
  bikeinfo.value = data;
};

// 關鍵字搜尋
const searchAr = ref('');
const bikeinfoFilterBySearchAr = computed(() => {
  return bikeinfo.value.filter((item) => item.ar.includes(searchAr.value));
});

// 關鍵字Highlight
const heightligthAr = (ar) => {
  if (searchAr.value) {
    return ar.replace(
      searchAr.value,
      `<span class="text-red-500 font-bold">${searchAr.value}</span>`
    );
  } else {
    return ar;
  }
};

// 目前排序
const currentSort = ref('');
// Desc 高到低, Asc 低到高
const isSortDesc = ref(null);
const sortBikeInfo = (type) => {
  const originType = currentSort.value;
  currentSort.value = type;

  // 第一次執行
  if (isSortDesc.value === null) {
    isSortDesc.value = true;
  } else {
    // 判斷是否為同一個欄位
    // 是的話切換排序
    // 不是的話預設為降冪排序
    if (originType === currentSort.value) {
      isSortDesc.value = !isSortDesc.value;
    } else {
      isSortDesc.value = true;
    }
  }
};

// 排序後的資料
const bikeinfoFilterBySort = computed(() => {
  let result = [...bikeinfoFilterBySearchAr.value];
  if (!currentSort.value) {
    return result;
  }

  return isSortDesc.value
    ? result.sort((a, b) => b[currentSort.value] - a[currentSort.value])
    : result.sort((a, b) => a[currentSort.value] - b[currentSort.value]);
});

// 當前頁數
const currentPage = ref(1);
// 總頁數
const bikeinfoFilterBySortTotal = computed(() => bikeinfoFilterBySort.value.length);

// 每頁顯示10筆資料
const rows = ref(20);
const bikeinfoFilterBySortSliced = computed(() => {
  let result = [];
  const start = (currentPage.value - 1) * rows.value;
  const end =
    start + rows.value <= bikeinfoFilterBySort.value.length
      ? start + rows.value
      : bikeinfoFilterBySort.value.length;
  result = [...bikeinfoFilterBySort.value.slice(start, end)];
  return result;
});

// 切換頁數
const changePage = (page) => {
  console.log('changePage', page);
  currentPage.value = page;
};

// 監聽搜尋關鍵字=>重置頁數
const isReset = ref(false);
watch(searchAr, () => {
  if (searchAr.value) {
    isReset.value = true;
  } else {
    isReset.value = false;
  }
});

onMounted(async () => {
  await getBikeInfo();
  initFlowbite();
});
</script>

<template>
  <div class="container mx-auto px-4">
    <!--top bar start-->
    <nav class="bg-white w-full top-0 start-0 border-b border-gray-300 mb-2">
      <div class="max-w-screen-xl flex flex-wrap items-center justify-between mx-auto p-4">
        <a href="https://flowbite.com/" class="flex items-center space-x-3 rtl:space-x-reverse">
          <img src="https://flowbite.com/docs/images/logo.svg" class="h-8" alt="Flowbite Logo" />
          <span class="self-center text-2xl font-semibold whitespace-nowrap dark:text-white"
            >YouBike範本</span
          >
        </a>
        <div class="flex md:order-2 space-x-3 md:space-x-0 rtl:space-x-reverse">
          <div class="mr-2">
            <InputText v-model="searchAr" placeholder="輸入站點地址"></InputText>
          </div>
          <button
            type="submit"
            class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-full sm:w-auto px-5 py-2.5 text-center cursor-pointer"
          >
            查詢
          </button>
        </div>
      </div>
    </nav>
    <!--top bar end-->

    <!--datatable start-->
    <div class="overflow-x-auto shadow-md">
      <table class="w-full text-sm text-left rtl:text-right text-gray-500 table-fixed">
        <thead class="text-white uppercase bg-blue-600 dark:text-white">
          <tr>
            <th class="font-semibold text-center p-2">站點編號</th>
            <th class="font-semibold text-center p-2">站點名稱</th>
            <th class="font-semibold text-center p-2">站點所在區域</th>
            <th class="font-semibold text-center p-2">站點地址</th>
            <th class="font-semibold text-center p-2">
              <div class="flex items-center justify-center">
                <span class="mr-1">總車位數量</span>
                <span class="cursor-pointer" @click="sortBikeInfo('total')">
                  <svg
                    class="w-[16px] h-[16px] text-white"
                    aria-hidden="true"
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    :fill="
                      currentSort === 'total' && isSortDesc === false ? 'currentColor' : 'none'
                    "
                    viewBox="0 0 24 24"
                  >
                    <path
                      v-if="currentSort === 'total' && isSortDesc === false"
                      fill-rule="evenodd"
                      d="M5.575 13.729C4.501 15.033 5.43 17 7.12 17h9.762c1.69 0 2.618-1.967 1.544-3.271l-4.881-5.927a2 2 0 0 0-3.088 0l-4.88 5.927Z"
                      clip-rule="evenodd"
                    />
                    <path
                      v-else
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M16.881 16H7.119a1 1 0 0 1-.772-1.636l4.881-5.927a1 1 0 0 1 1.544 0l4.88 5.927a1 1 0 0 1-.77 1.636Z"
                    />
                  </svg>

                  <svg
                    class="w-[16px] h-[16px] text-white"
                    aria-hidden="true"
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    :fill="currentSort === 'total' && isSortDesc === true ? 'currentColor' : 'none'"
                    viewBox="0 0 24 24"
                  >
                    <path
                      v-if="currentSort === 'total' && isSortDesc === true"
                      fill-rule="evenodd"
                      d="M18.425 10.271C19.499 8.967 18.57 7 16.88 7H7.12c-1.69 0-2.618 1.967-1.544 3.271l4.881 5.927a2 2 0 0 0 3.088 0l4.88-5.927Z"
                      clip-rule="evenodd"
                    />
                    <path
                      v-else
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M7.119 8h9.762a1 1 0 0 1 .772 1.636l-4.881 5.927a1 1 0 0 1-1.544 0l-4.88-5.927A1 1 0 0 1 7.118 8Z"
                    />
                  </svg>
                </span>
              </div>
            </th>
            <th class="font-semibold text-center p-2">
              <div class="flex items-center justify-center">
                <span class="mr-1">可租借的腳踏車數量</span>
                <span class="cursor-pointer" @click="sortBikeInfo('available_rent_bikes')">
                  <svg
                    class="w-[16px] h-[16px] text-white"
                    aria-hidden="true"
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    :fill="
                      currentSort === 'available_rent_bikes' && isSortDesc === false
                        ? 'currentColor'
                        : 'none'
                    "
                    viewBox="0 0 24 24"
                  >
                    <path
                      v-if="currentSort === 'available_rent_bikes' && isSortDesc === false"
                      fill-rule="evenodd"
                      d="M5.575 13.729C4.501 15.033 5.43 17 7.12 17h9.762c1.69 0 2.618-1.967 1.544-3.271l-4.881-5.927a2 2 0 0 0-3.088 0l-4.88 5.927Z"
                      clip-rule="evenodd"
                    />
                    <path
                      v-else
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M16.881 16H7.119a1 1 0 0 1-.772-1.636l4.881-5.927a1 1 0 0 1 1.544 0l4.88 5.927a1 1 0 0 1-.77 1.636Z"
                    />
                  </svg>
                  <svg
                    class="w-[16px] h-[16px] text-white"
                    aria-hidden="true"
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    :fill="
                      currentSort === 'available_rent_bikes' && isSortDesc === true
                        ? 'currentColor'
                        : 'none'
                    "
                    viewBox="0 0 24 24"
                  >
                    <path
                      v-if="currentSort === 'available_rent_bikes' && isSortDesc === true"
                      fill-rule="evenodd"
                      d="M18.425 10.271C19.499 8.967 18.57 7 16.88 7H7.12c-1.69 0-2.618 1.967-1.544 3.271l4.881 5.927a2 2 0 0 0 3.088 0l4.88-5.927Z"
                      clip-rule="evenodd"
                    />
                    <path
                      v-else
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M7.119 8h9.762a1 1 0 0 1 .772 1.636l-4.881 5.927a1 1 0 0 1-1.544 0l-4.88-5.927A1 1 0 0 1 7.118 8Z"
                    />
                  </svg>
                </span>
              </div>
            </th>
            <th class="font-semibold text-center p-2">站點緯度</th>
            <th class="font-semibold text-center p-2">站點經度</th>
            <th class="font-semibold text-center p-2">可歸還的腳踏車數量</th>
          </tr>
        </thead>
        <tbody>
          <tr
            class="odd:bg-white even:bg-gray-100 border-b hover:bg-blue-100"
            v-for="(item, index) in bikeinfoFilterBySortSliced"
            :key="item.sno"
          >
            <td class="p-2 text-center">{{ item.sno }}</td>
            <td class="p-2 text-center">{{ item.sna }}</td>
            <td class="p-2 text-center">{{ item.sarea }}</td>
            <td class="p-2 text-center" v-html="heightligthAr(item.ar)"></td>
            <td class="p-2 text-center">{{ item.total }}</td>
            <td class="p-2 text-center">{{ item.available_rent_bikes }}</td>
            <td class="p-2 text-center">{{ item.latitude }}</td>
            <td class="p-2 text-center">{{ item.longitude }}</td>
            <td class="p-2 text-center">{{ item.available_return_bikes }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <!--datatable end-->

    <!--pagination start-->
    <Paginator
      :totalRecords="bikeinfoFilterBySortTotal"
      :rows="rows"
      @page="changePage"
      :isReset="isReset"
    ></Paginator>
    <!--pagination end-->
  </div>
</template>
