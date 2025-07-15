

<template>

  <label for="search">搜尋: </label>
  <label class="input">
    <svg class="h-[1em] opacity-50" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
      <g
        stroke-linejoin="round"
        stroke-linecap="round"
        stroke-width="2.5"
        fill="none"
        stroke="currentColor"
      >
        <circle cx="11" cy="11" r="8"></circle>
        <path d="m21 21-4.3-4.3"></path>
      </g>
    </svg>
    <input type="search" class="grow" placeholder="Search" v-model="inputName" @keyup="filter()" @keyup.enter="filter()"/>
  </label>
  <button @click="filter()" class="btn ">搜尋</button>
  <button @click="showCollectionToggle()" class="btn "> 
    <span v-if="!isShowCollectionOnly">我的收藏</span>
    <span v-if="isShowCollectionOnly">顯示全部</span>
  </button>
  <br>
  <table class="table">
    <!-- head -->
    <thead>
      <tr>
        <th>站名</th>
        <th>區</th>
        <th>可租借</th>
        <th>可歸還</th>
        <th>操作</th>
      </tr>
    </thead>
    <tbody>
      <!-- row 1 -->
      <tr v-for="station in display_stations">
        <th>{{ station.sna }}</th>
        <th>{{ station.sarea }}</th>
        <th>{{ station.available_rent_bikes }}</th>
        <th>{{ station.available_return_bikes }}</th>
        <th> 
          <button class="btn " onclick="my_modal_1.showModal()" @click="showModal(station)">
            詳細
          </button>
          <button class="btn btn-outline btn-secondary" @click="add(station.sno)" v-if="!station.isCollected"> 
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="size-[1.2em]"><path stroke-linecap="round" stroke-linejoin="round" d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12Z" /></svg>
            收藏
          </button>
          <button class="btn btn-outline btn-secondary" @click="remove(station.sno)" v-if="station.isCollected">
            <svg xmlns="http://www.w3.org/2000/svg" fill="oklch(65% 0.241 354.308)" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="size-[1.2em]"><path stroke-linecap="round" stroke-linejoin="round" d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12Z" /></svg>
            已收藏
          </button>
        </th>
      </tr>
    </tbody>
  </table>
  <dialog id="my_modal_1" class="modal">
    <div class="modal-box">
      <h3 class="text-lg font-bold mb-2">{{ currentStation.sna }}</h3>
      <!-- 站點詳細資訊 -->
      <div class="mb-2 space-y-1">
        <div><span class="font-semibold text-gray-700">區域：</span><span class="ml-2">{{ currentStation.sarea }}</span></div>
        <div><span class="font-semibold text-gray-700">地址：</span><span class="ml-2">{{ currentStation.ar }}</span></div>
        <div><span class="font-semibold text-gray-700">可租借車輛數：</span><span class="ml-2">{{ currentStation.available_rent_bikes }}</span></div>
        <div><span class="font-semibold text-gray-700">可歸還車輛數：</span><span class="ml-2">{{ currentStation.available_return_bikes }}</span></div>
        <div><span class="font-semibold text-gray-700">總格數：</span><span class="ml-2">{{ currentStation.Quantity }}</span></div>
        <div><span class="font-semibold text-gray-700">經度：</span><span class="ml-2">{{ currentStation.longitude }}</span></div>
        <div><span class="font-semibold text-gray-700">緯度：</span><span class="ml-2">{{ currentStation.latitude }}</span></div>
        <div><span class="font-semibold text-gray-700">是否已收藏：</span><span class="ml-2"><span v-if="currentStation.isCollected">是</span><span v-else>否</span></span></div>
        <div><span class="font-semibold text-gray-700">資料更新時間：</span><span class="ml-2">{{ currentStation.updateTime }}</span></div>
      </div>
      <div class="modal-action">
        <form method="dialog">
          <button class="btn">關閉</button>
        </form>
      </div>
    </div>
  </dialog>
</template>

<script setup>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';

  const stations = ref([]);
  const display_stations = ref([]);
  const inputName = ref('');
  const myCollection = ref({});
  const isShowCollectionOnly = ref(false);
  const currentStation = ref({}); // 用於存放目前要顯示的站點資訊

  onMounted(() => {
    const storge = localStorage.getItem('myCollection');
    console.log(storge);
    if(storge){
      myCollection.value = JSON.parse(storge);
    }
    fetch_youbike_information();
  });
  const fetch_youbike_information = async () => {
    const { data } = await axios.get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json');
    // 將每個站點物件加上 isCollected: false
    stations.value = data.map(station => ({
      ...station,
      isCollected: myCollection.value[station.sno] 
    }));
    
    display_stations.value = stations.value;
  }

  const showCollectionToggle = () => {
    isShowCollectionOnly.value = !isShowCollectionOnly.value;
    filter()
  }

  const filter = () => {
    display_stations.value = stations.value.filter( (s) => {
      return s.sna.toLowerCase().includes(inputName.value.toLowerCase()) || s.sarea.toLowerCase().includes(inputName.value.toLowerCase())
    });
    if(isShowCollectionOnly.value){
      display_stations.value = display_stations.value.filter(s => s.isCollected);
    }
  }

  // 新增收藏，並根據 id (sno) 找出對應站點
  const add = async (id) => {
    myCollection.value[id] = true;
    const foundStation = stations.value.find(s => s.sno === id);
    foundStation.isCollected = true;
    updateStorge();
    filter();
  }
  // 移除收藏
  const remove = async (id) => {
    delete myCollection.value[id];
    const foundStation = stations.value.find(s => s.sno === id);
    foundStation.isCollected = false;
    updateStorge();
    filter();
  }
  const updateStorge = () => {
    localStorage.setItem('myCollection', JSON.stringify(myCollection.value));
    console.log(myCollection.value)
  }

  // 顯示詳細資訊 modal
  const showModal = (info) => {
    currentStation.value = info;
  }

</script>



<style scoped>

</style>
