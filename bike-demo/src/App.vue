<script setup>
import { ref } from 'vue';
import axios from 'axios';
import Paginate from 'vuejs-paginate-next';
const dataArray = ref([]);
const showData = ref([]);
const searchData = ref([]);
const searchStr = ref('');
const pages = ref(10);
const current = ref(1);
const sortStr = ref('*️⃣');
const sortStr2 = ref('*️⃣');
const isShowTd = ref(false);

//取得api資料
axios
  .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
  .then(function (res) {
    dataArray.value = res.data;

    for (let i = 0; i < 20; i++) {
      showData.value.push(dataArray.value[i]);
    }
    pages.value = Math.ceil(dataArray.value.length / 20);
  })
  .catch(function (error) {
    console.log(error);
  });

function totalSort() {
  //第一次排序，由小到大
  if (sortStr.value == '*️⃣' || sortStr.value == '🔽') {
    console.log('小到大');
    //小到大
    dataArray.value.sort((a, b) => a.total - b.total);
    showData.value = dataArray.value;
    sortStr.value = '🔼';
    sortStr2.value = '*️⃣';

    //排序完，如果有查詢input有資料，再查詢一次
    if (searchStr.value) {
      doSearch();
    }
  } else {
    //大到小
    console.log('大到小');
    dataArray.value.sort((a, b) => b.total - a.total);
    showData.value = dataArray.value;
    sortStr.value = '🔽';
    sortStr2.value = '*️⃣';
    //排序完，如果有查詢input有資料，再查詢一次
    if (searchStr.value) {
      doSearch();
    }
  }
  //分頁
  doChangePage();
}

function vailableRentBikesSort() {
  //第一次排序，由小到大
  if (sortStr2.value == '*️⃣' || sortStr2.value == '🔽') {
    console.log('小到大');
    //小到大
    dataArray.value.sort((a, b) => a.available_rent_bikes - b.available_rent_bikes);
    showData.value = dataArray.value;
    sortStr2.value = '🔼';
    sortStr.value = '*️⃣';

    //排序完，如果有查詢input有資料，再查詢一次
    if (searchStr.value) {
      doSearch();
    }
  } else {
    //大到小
    console.log('大到小');
    dataArray.value.sort((a, b) => b.available_rent_bikes - a.available_rent_bikes);
    showData.value = dataArray.value;
    sortStr2.value = '🔽';
    sortStr.value = '*️⃣';
    //排序完，如果有查詢input有資料，再查詢一次
    if (searchStr.value) {
      doSearch();
    }
  }
  //分頁
  doChangePage();
}

function doSearch() {
  showData.value = [];
  searchData.value = [];
  if (searchStr.value == null || searchStr.value == '') {
    isShowTd.value = false;
    showData.value = dataArray.value;
    pages.value = Math.ceil(dataArray.value.length / 20);
  } else {
    isShowTd.value = true;
    searchStr.value = searchStr.value.trim();

    searchData.value = dataArray.value.filter((data) => data.ar.includes(searchStr.value));

    pages.value = Math.ceil(searchData.value.length / 20);
    current.value = current.value > pages.value ? pages.value : current.value;
    for (let i = 0; i < 20; i++) {
      if (i >= searchData.value.length) {
        break;
      }

      showData.value.push(searchData.value[i]);
    }
  }
  doChangePage();
}

function doChangePage() {
  showData.value = [];
  const prevIndex = 20 + (current.value - 2) * 20;
  const index = 20 + (current.value - 1) * 20;
  //判斷是否有搜尋條件，選擇要抓searchData還是dataArray
  if (searchStr.value == null || searchStr.value == '') {
    if (index > dataArray.value.length) {
      showData.value = dataArray.value.slice(prevIndex, dataArray.value.length);
    } else {
      showData.value = dataArray.value.slice(prevIndex, index);
    }
  } else {
    if (index > searchData.value.length) {
      showData.value = searchData.value.slice(prevIndex, dataArray.value.length);
    } else {
      showData.value = searchData.value.slice(prevIndex, index);
    }
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
      :click-handler="doChangePage"
      :first-last-button="true"
    ></Paginate>
  </div>
</template>

<style scoped></style>
