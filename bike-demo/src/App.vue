<script setup>
import { ref } from 'vue'
import axios from 'axios'
import Paginate from 'vuejs-paginate-next'
const dataArray = ref([])
const showData = ref([])
const searchData = ref([])
const searchStr = ref('')
const pages = ref(10)
const current = ref(1)

//取得api資料
axios
  .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
  .then(function (res) {
    for (let i = 0; i < res.data.length; i++) {
      dataArray.value.push(res.data[i])
    }
    for (let i = 0; i < 20; i++) {
      showData.value.push(dataArray.value[i])
    }

    pages.value = Math.ceil(dataArray.value.length / 20)
  })
  .catch(function (error) {
    console.log(error)
  })

function totalSort() {
  if (showData.value[0].total > showData.value[1].total) {
    //小到大
    showData.value.sort((a, b) => a.total - b.total)
  } else {
    //大到小
    showData.value.sort((a, b) => b.total - a.total)
  }
}

function vailableRentBikesSort() {
  if (showData.value[0].available_rent_bikes > showData.value[1].available_rent_bikes) {
    //小到大
    showData.value.sort((a, b) => a.available_rent_bikes - b.available_rent_bikes)
  } else {
    //大到小
    showData.value.sort((a, b) => b.available_rent_bikes - a.available_rent_bikes)
  }
}

function doSearch() {
  current.value = 1
  showData.value = []
  searchData.value = []

  if (searchStr.value == null || searchStr.value == '') {
    dataArray.value = []
    axios
      .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
      .then(function (res) {
        for (let i = 0; i < res.data.length; i++) {
          dataArray.value.push(res.data[i])
        }
        for (let i = 0; i < 20; i++) {
          showData.value.push(dataArray.value[i])
        }

        pages.value = Math.ceil(dataArray.value.length / 20)
      })
      .catch(function (error) {
        console.log(error)
      })
    doChangePage()
  } else {
    searchStr.value = searchStr.value.trim()
    for (let i = 0; i < dataArray.value.length; i++) {
      if (dataArray.value[i].ar.includes(searchStr.value)) {
        searchData.value.push(dataArray.value[i])
      }
    }

    for (let i = 0; i < searchData.value.length; i++) {
      let idx = searchData.value[i].ar.indexOf(searchStr.value)
      let Redstr = searchData.value[i].ar.substr(idx, searchStr.value.length)
      let frontStr = searchData.value[i].ar.substr(0, idx)
      let backStr = searchData.value[i].ar.substr(
        idx + searchStr.value.length,
        searchData.value[i].ar.length
      )
      let str =
        '<span>' +
        frontStr +
        '</span>' +
        ' <span style="color:	#FF0000">' +
        Redstr +
        '</span>' +
        '<span>' +
        backStr +
        '</span>'

      searchData.value[i].ar = str
    }

    pages.value = Math.ceil(searchData.value.length / 20)

    for (let i = 0; i < 20; i++) {
      if (i >= searchData.value.length) {
        break
      }

      showData.value.push(searchData.value[i])
    }
  }
}
function doChangePage() {
  showData.value = []
  const prevIndex = 20 + (current.value - 2) * 20
  const index = 20 + (current.value - 1) * 20
  //判斷是否有搜尋條件，選擇要抓searchData還是dataArray
  if (searchStr.value == null || searchStr.value == '') {
    for (let i = prevIndex; i < index; i++) {
      if (i >= dataArray.value.length) {
        break
      }

      showData.value.push(dataArray.value[i])
    }
  } else {
    for (let i = prevIndex; i < index; i++) {
      if (i >= searchData.value.length) {
        break
      }
      showData.value.push(searchData.value[i])
    }
  }
}
</script>

<template>
  <nav class="navbar navbar-expand-lg bg-body-tertiary" style="padding-top: 2%">
    <div class="container-fluid">
      <a class="navbar-brand" href="#">Bike Demo</a>

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
  <div style="padding: 2%">
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
        <tr v-for="data in showData" :key="data">
          <th scope="row">{{ data.sno }}</th>
          <td>{{ data.sna }}</td>
          <td>{{ data.sarea }}</td>
          <td v-html="data.ar"></td>
          <td>{{ data.total }}</td>
          <td>{{ data.available_rent_bikes }}</td>
          <td>{{ data.latitude }}</td>
          <td>{{ data.longitude }}</td>
          <td>{{ data.available_return_bikes }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped></style>
