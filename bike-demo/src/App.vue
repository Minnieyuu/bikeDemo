<script setup>
import { ref, watch, onMounted, computed } from 'vue';
import axios from 'axios';
import Paginate from 'vuejs-paginate-next';

const dataArray = ref([]);
const showData = ref([]);
const filterData = ref([]);
const searchStr = ref('');
const current = ref(1);
const sortStr = ref('*️⃣');
const sortStr2 = ref('*️⃣');
const isShowTd = ref(false);

//取得api資料
onMounted(async () => {
  const response = await axios.get(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  dataArray.value = JSON.parse(JSON.stringify(response.data));
  //filterData初始值
  filterData.value = JSON.parse(JSON.stringify(response.data));
});

//計算頁數
const pages = computed(() => Math.ceil(filterData.value.length / 20));

watch(
  () => filterData.value,
  () => {
    console.log('doWatch');

    showData.value = [];
    //啟始點
    const prevIndex = 20 + (current.value - 2) * 20;
    //結束點
    const index = 20 + (current.value - 1) * 20;
    //如果結束點超過陣列長度，取陣列長度
    if (index > dataArray.value.length) {
      showData.value = [...filterData.value.slice(prevIndex, filterData.value.length)];
    } else {
      showData.value = [...filterData.value.slice(prevIndex, index)];
    }
    console.log(showData.value);
  }
);

watch(
  () => current.value,
  () => {
    showData.value = [];
    const prevIndex = 20 + (current.value - 2) * 20;
    const index = 20 + (current.value - 1) * 20;

    if (index > dataArray.value.length) {
      showData.value = [...filterData.value.slice(prevIndex, filterData.value.length)];
    } else {
      showData.value = [...filterData.value.slice(prevIndex, index)];
    }
  }
);

function totalSort() {
  if (sortStr.value == '*️⃣' || sortStr.value == '🔽') {
    console.log('小到大');
    //小到大
    filterData.value = [...filterData.value.sort((a, b) => a.total - b.total)];
    sortStr.value = '🔼';
    sortStr2.value = '*️⃣';
  } else {
    //大到小
    console.log('大到小');
    filterData.value = [...filterData.value.sort((a, b) => b.total - a.total)];
    sortStr.value = '🔽';
    sortStr2.value = '*️⃣';
  }
}

function vailableRentBikesSort() {
  if (sortStr2.value == '*️⃣' || sortStr2.value == '🔽') {
    console.log('小到大');
    //小到大
    filterData.value = [
      ...filterData.value.sort((a, b) => a.available_rent_bikes - b.available_rent_bikes)
    ];
    sortStr2.value = '🔼';
    sortStr.value = '*️⃣';
  } else {
    //大到小
    console.log('大到小');
    filterData.value = [
      ...filterData.value.sort((a, b) => b.available_rent_bikes - a.available_rent_bikes)
    ];
    sortStr2.value = '🔽';
    sortStr.value = '*️⃣';
  }
}

function doSearch() {
  showData.value = [];
  filterData.value = [];
  if (searchStr.value == null || searchStr.value == '') {
    isShowTd.value = false;
    filterData.value = JSON.parse(JSON.stringify(dataArray.value));
  } else {
    isShowTd.value = true;
    searchStr.value = searchStr.value.trim();
    filterData.value = JSON.parse(
      JSON.stringify(dataArray.value.filter((data) => data.ar.includes(searchStr.value)))
    );
    current.value = 1;
  }
}
</script>

<template>
  <nav class="navbar navbar-expand-lg bg-body-tertiary">
    <div class="container-fluid">
      <img
        style="padding: 2px"
        src="/src/assets/photo/bikeIcon.png"
        alt=""
        width="40"
        height="40"
      />
      <a class="navbar-brand" href="#" style="padding-left: 5px">Bike Demo</a>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <input
          @change="doSearch"
          class="form-control me-2"
          placeholder="Search"
          v-model="searchStr"
          aria-label="Search"
        />
        <button class="btn btn-outline-success" @click="doSearch">Search</button>
      </div>
    </div>
  </nav>
  <div style="padding-top: 10px">
    <table class="table table-hover">
      <thead>
        <tr class="table-primary">
          <th scope="col">站點編號</th>
          <th scope="col">站點名稱</th>
          <th scope="col">站點所在區域</th>
          <th scope="col">站點地址</th>
          <th @click="totalSort" scope="col">
            總車位數量 <span>{{ sortStr }}</span>
          </th>
          <th @click="vailableRentBikesSort" scope="col">
            可租借的腳踏車數量 <span>{{ sortStr2 }}</span>
          </th>
          <th scope="col">站點緯度</th>
          <th scope="col">站點經度</th>
          <th scope="col">可歸還的腳踏車數量</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="data in showData" :key="data">
          <th scope="row">{{ data.sno }}</th>
          <td>{{ data.sna }}</td>
          <td>{{ data.sarea }}</td>
          <td v-show="!isShowTd">{{ data.ar }}</td>
          <td v-show="isShowTd">
            <span>{{ data.ar.substr(0, data.ar.indexOf(searchStr)) }}</span
            ><span style="color: red">{{
              data.ar.substr(data.ar.indexOf(searchStr), searchStr.length)
            }}</span
            ><span>{{
              data.ar.substr(data.ar.indexOf(searchStr) + searchStr.length, data.ar.length)
            }}</span>
          </td>
          <td>{{ data.total }}</td>
          <td>{{ data.available_rent_bikes }}</td>
          <td>{{ data.latitude }}</td>
          <td>{{ data.longitude }}</td>
          <td>{{ data.available_return_bikes }}</td>
        </tr>
      </tbody>
    </table>
    <Paginate
      class="justify-content-center"
      first-button-text="&lt;&lt;"
      last-button-text="&gt;&gt;"
      prev-text="&lt;"
      next-text="&gt;"
      :page-count="pages"
      :initial-page="current"
      v-model="current"
      :first-last-button="true"
    ></Paginate>
  </div>
</template>

<style scoped></style>
