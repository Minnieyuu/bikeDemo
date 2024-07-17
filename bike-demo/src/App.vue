<script setup>
import { ref } from 'vue'
import axios from 'axios'

const dataArray = ref([])
const searchData = ref([])
const isSearchShow = ref(true)

const searchStr = ref('')

axios
  .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
  .then(function (res) {
    // dataArray.value.push(res.data[1000].total)

    for (let i = 0; i < res.data.length; i++) {
      dataArray.value.push(res.data[i])
    }
    console.log(dataArray.value)
  })
  .catch(function (error) {
    console.log(error)
  })

function totalSort() {
  if (searchStr.value == null || searchStr.value == '') {
    if (dataArray.value[0].total > dataArray.value[1].total) {
      //小到大
      dataArray.value.sort((a, b) => a.total - b.total)
    } else {
      //大到小
      dataArray.value.sort((a, b) => b.total - a.total)
    }
  } else {
    if (searchData.value[0].total > searchData.value[1].total) {
      //小到大
      searchData.value.sort((a, b) => a.total - b.total)
    } else {
      //大到小
      searchData.value.sort((a, b) => b.total - a.total)
    }
  }
}

function vailableRentBikesSort() {
  if (searchStr.value == null || searchStr.value == '') {
    if (dataArray.value[0].available_rent_bikes > dataArray.value[1].available_rent_bikes) {
      //小到大
      dataArray.value.sort((a, b) => a.available_rent_bikes - b.available_rent_bikes)
    } else {
      //大到小
      dataArray.value.sort((a, b) => b.available_rent_bikes - a.available_rent_bikes)
    }
  } else {
    if (searchData.value[0].available_rent_bikes > searchData.value[1].available_rent_bikes) {
      //小到大
      searchData.value.sort((a, b) => a.available_rent_bikes - b.available_rent_bikes)
    } else {
      //大到小
      searchData.value.sort((a, b) => b.available_rent_bikes - a.available_rent_bikes)
    }
  }
}

function doSearch() {
  if (searchStr.value == null || searchStr.value == '') {
    isSearchShow.value = true
  }
  searchData.value = []
  isSearchShow.value = false
  for (let i = 0; i < dataArray.value.length; i++) {
    if (dataArray.value[i].ar.includes(searchStr.value)) {
      searchData.value.push(dataArray.value[i])
    }
  }
}
</script>

<template>
  <!-- <Pagination page-count="10" page-range="1" init-page="1" margin-pages="2"> </Pagination> -->
  <div class="input-group input-group-sm mb-1" style="margin: 50px">
    <span class="input-group-text" id="inputGroup-sizing-sm">搜尋</span>
    <input
      v-model="searchStr"
      type="text"
      class="form-control"
      aria-label="Sizing example input"
      aria-describedby="inputGroup-sizing-sm"
    />
  </div>
  <button @click="doSearch">search</button>
  <table class="table table-hover">
    <thead>
      <tr class="table-primary">
        <th scope="col">站點編號</th>
        <th scope="col">站點名稱</th>
        <th scope="col">站點所在區域</th>
        <th scope="col">站點地址</th>
        <th @click="totalSort" scope="col">總車位數量(排序)</th>
        <th @click="vailableRentBikesSort" scope="col">可租借的腳踏車數量(排序)</th>
        <th scope="col">站點緯度</th>
        <th scope="col">站點經度</th>
        <th scope="col">可歸還的腳踏車數量</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="data in dataArray" :key="data" v-show="isSearchShow">
        <th scope="row">{{ data.sno }}</th>
        <td>{{ data.sna }}</td>
        <td>{{ data.sarea }}</td>
        <td>{{ data.ar }}</td>
        <td>{{ data.total }}</td>
        <td>{{ data.available_rent_bikes }}</td>
        <td>{{ data.latitude }}</td>
        <td>{{ data.longitude }}</td>
        <td>{{ data.available_return_bikes }}</td>
      </tr>
      <tr v-for="data in searchData" :key="data" v-show="!isSearchShow">
        <th scope="row">{{ data.sno }}</th>
        <td>{{ data.sna }}</td>
        <td>{{ data.sarea }}</td>
        <td>{{ data.ar }}</td>
        <td>{{ data.total }}</td>
        <td>{{ data.available_rent_bikes }}</td>
        <td>{{ data.latitude }}</td>
        <td>{{ data.longitude }}</td>
        <td>{{ data.available_return_bikes }}</td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped></style>
