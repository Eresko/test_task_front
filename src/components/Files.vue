<script setup>
import { ref,watchEffect,onMounted,reactive,computed } from 'vue'
import { mapGetters, mapActions } from "vuex";
import { useCookies } from "vue3-cookies";
import { debouncedWatch } from '@vueuse/core'
const { cookies } = useCookies();
import axios from 'axios'
import VFlexPagination from './VFlexPagination.vue'
defineProps({
  msg: String,
})
const flagQuery = ref(false);
const shops  = ref([]);
const filters = ref({
  current_page: 1,
  per_page: 10,
  total: 0,
  last_page: 0,
})
const contents = ref();
const searchText = ref("");
const files = ref([]);
const detail = ref(false)
const page = reactive({current: 1,old:1})
debouncedWatch(searchText, () => getFiles(), { debounce: 500 })
const file = ref();
const fileUpload = () => {
  file.value = document.getElementById("fileUploadNow").files[0];
}
const pushFile = async () => {
  flagQuery.value = true;
  let formData = new FormData();
  formData.append('file', file.value);
  await axios.post('https://api-test.zedform.ru/api/nomenclature/upload-file', formData,
      {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(function(){

    getFiles();



  }).catch(function(){
    console.log('FAILURE!!');
  });
  console.log("PUSH FILE");
  flagQuery.value = false;
}
const getFiles = async () => {
     flagQuery.value = true;
     await axios.get('https://api-test.zedform.ru/api/nomenclature/list?page=' + page.current + '&search=' + searchText.value ).then( response  => {
          files.value = response.data.result.data
          filters.value.current_page = response.data.result.currentPage
          filters.value.per_page = response.data.result.perPage
          filters.value.total = response.data.result.total
          filters.value.last_page = response.data.result.lastPage
          flagQuery.value = false;
    }).catch( res => {
       cookies.set('token',null);

     })
  document.getElementById("formStart")?.focus()
}

const getContentsFile = async (id) => {
  flagQuery.value = true;
  await axios.get('https://api-test.zedform.ru/api/nomenclature/get/' + id ).then( response  => {
    contents.value = response.data.result.file_element
    flagQuery.value = false;
    detail.value =true;
  }).catch( res => {
    cookies.set('token',null);

  })
}

watchEffect(() => {
    if (page.current != page.old) {
      page.old = page.current;
      getFiles();
      document.getElementById("formStart")?.focus()
    }
 })

onMounted( async () => {
  await getFiles();
})

const show = (item) =>{
  let mas = JSON.parse(item)
  let dir ="";
  mas.forEach( n => {
    dir +="<div style='width:"+ 100/mas.length +"%'>" + n + "</div>";
  })
  return dir
}
</script>

<template>

  <div style="width: 1920px;" v-if="flagQuery">
    <img src="/img/gg.gif">
  </div>
  <div style="width: 1920px;" >
    <div class="block_head" v-if="!flagQuery && detail== false">
      <div class="search_input">
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" height="40" viewBox="0 -960 960 960" width="40"><path d="M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z"/></svg>
        <input class="search" v-model="searchText" ref="formStart" id="formStart">
      </div>
      <div class="search_block">
        <div class="search_input">
          <input type="file"   id="fileUploadNow" @change="fileUpload" accept=".csv" class="push_file"/>
          <button  @click="pushFile" value="upload" >UPLOAD</button>
        </div>
      </div>
    </div>
    <div v-for="(file,key) in files" class="block" @click="getContentsFile(file.id)" v-if="!flagQuery && detail== false">
      <div class="block__key">
        {{file.id}}
      </div>
      <div class="block__name">
        {{ file.name }}
      </div>
      <div class="block__code">
        {{ file.created_at }}
      </div>
    </div>
    <div v-else-if="detail == true && !flagQuery">
      <div v-for="item in contents" class="parent_content">
        <div v-html="show(item.line)" class="child_content"></div>
      </div>
    </div>
    <VFlexPagination
        v-if="filters.last_page > 1 && detail == false"
        :total-items="filters.total"
        :item-per-page="filters.per_page"
        :current-page="filters.current_page"
        :max-links-displayed="3"
        @update:page="page.current = $event"
    />
  </div>
</template>

<style lang="scss" scoped>
#app {
  max-width: 1920px!important;
}
.push_file {
  font-size: 29px;
}
.parent_content {
  display: flex;
  flex-direction: column;
  width: 100%;
}
.child_content {
  display: flex;
  flex-direction: row;
  width: 100%;
}
@keyframes ch-color-1 {
  from {
    fill: black;
  }
  to {
    fill: #646cff;
  }
}
.icon {
  margin-left: 5px;
  fill:white;
}
.search {
  outline:none;
  width: 300px;
  border: 0px;
  background-color: #1a1a1a;
  margin-left: 10px;
  height: 40px;
  font-size: 20px;
}
.block_head {
  display: flex;
  flex-direction: row;
  margin-left: 35px;
  height: 25px;
  margin-bottom: 50px;
  justify-content: space-around;
}
.search_block {


}
.search_input {
  display: flex;
  flex-direction: row;
  width: 450px;
  border-radius: 8px;
  border: 1px solid transparent;
  background-color: #1a1a1a;
  height: 40px;
}
.login {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  input {
    margin-top: 5px;
    max-width: 200px;
  }
  button {
    margin-top: 10px;
  }
}
.img-svg:hover {
  animation: ch-color-1 1s forwards;
}
.download {
  display: flex;
  justify-content: center;
  flex-direction: row;
  align-items: center;
}
.download_icon {
  color: #fff;
  cursor: pointer;
  border: 1px solid #242424;
}
.download_icon:hover {
  color: #646cff;
}
.block:hover {
  background-color: #646cff;
}
.block {
  cursor: pointer;
  margin-bottom: 4px;
  display: flex;
  justify-content: center;
  flex-direction: row;
  width: 100%;
  &__key {
    width: 15%;

   }
  &__name {
    width: 30%;
  }
  &__code {
    width: 40%;
  }
  &__cert {
    width: 40%;
  }
}

</style>