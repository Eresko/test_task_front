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
const searchText = ref("");
const page = reactive({current: 1,old:1})
const auth = ref(false);
const login = ref('');
const password = ref('');
debouncedWatch(searchText, () => getShop(), { debounce: 500 })
const test = async  () => {
  let token = cookies.get('token');
  const config = {
    headers:{
      "Content-type": "application/json",
      "Authorization": `Bearer 35464rtyrty64yry5y565656567`,
    }
  };
  let resp;
  await axios.get('https://api-cert.zedform.ru/api/age',config ).then( response  => {
    resp = response.data
  })


}
const getShop = async () => {

  let token = cookies.get('token');

  // if (token == null) {
  //   return;
  // }
  const config = {
    headers:{
      "Content-type": "application/json",
      "Authorization": `Bearer ${token}`,
      //"Authorization": `Bearer 123123`,

    }
  };
    let resp;
     flagQuery.value = true;
     await axios.get('https://api-cert.zedform.ru/api/shops?page=' + page.current + '&search=' + searchText.value,config ).then( response  => {
       resp = response.data
       shops.value = response.data.data
       filters.value.current_page = response.data.current_page
       filters.value.per_page = response.data.per_page
       filters.value.total = response.data.total
       filters.value.last_page = response.data.last_page
       flagQuery.value = false;
       document.getElementById("formStart")?.focus()
    }).catch( res => {
       cookies.set('token',null);

     })
  document.getElementById("formStart")?.focus()
}
const createCert = async (id) => {
  let resp;
  let token = cookies.get('token');
  const config = {
    headers:{
      "Content-type": "application/json",
      "Authorization": `Bearer ${token}`,
    }
  };
  flagQuery.value = true;
  await axios.get('https://api-cert.zedform.ru/api/create-cert/' + id, config).then( response  => {
    console.log(response.data);

    flagQuery.value = false;
    getShop();
  })
}

const authLogin = async () => {

  let playload = {
    login: login.value,
    password : password.value
  };
  await axios.post('https://api-cert.zedform.ru/api/login',playload).then( response  => {
    console.log(response.data);
    if (typeof response.data.token != 'undefined') {
      cookies.set('token',response.data.token);
      auth.value = true;
      document.location.href="/"
    }
  })
}
const authCheck = computed(() => {

  if (typeof cookies.get('token') == 'undefined') {
    return false
  }
  if (cookies.get('token') == 'null') {
    return false
  }
  return true
})
const input = ref<HTMLInputElement | null>(null);
watchEffect(() => {
  if (input.value) {
    console.log('123123');
    input.value.focus()
  }
})
watchEffect(() => {
    if (page.current != page.old) {
      page.old = page.current;
      getShop();
      document.getElementById("formStart")?.focus()
    }
 })

onMounted( async () => {
  await getShop();
  document.getElementById("formStart")?.focus()
})
const downloadExcel = (file) => {
  window.open('https://api-cert.zedform.ru/storage/' + file + '.tar.gz')
}

</script>

<template>
  <div style="width: 1100px;" v-if="authCheck == false">
    <div class="login">
      login <input type="text" v-model="login">
      password <input type="password" v-model="password">
      <button @click="authLogin">enter</button>
    </div>

  </div>
  <div style="width: 1100px;" v-else-if="flagQuery">
    <img src="/img/gg.gif">
  </div>
  <div style="width: 1100px;" v-else>
    <div class="search_block">
      <div class="search_input">
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 -960 960 960" width="24"><path d="M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z"/></svg>
        <input class="search" v-model="searchText" ref="formStart" id="formStart">
      </div>
    </div>
    <div v-for="(shop,key) in shops" class="block" >
      <div class="block__key">
        {{shop.id}}
      </div>
      <div class="block__name">
        {{ shop.name }}
      </div>
      <div class="block__code">
        {{ shop.code }}
      </div>
      <div class="block__cert" v-if="shop.cert == null">
      <button @click="createCert(shop.id)">create cert</button>
      </div>
      <div class="block__cert download" v-else>
        <div class="download_icon" @click="downloadExcel(shop.code)">
          <svg class="img-svg" xmlns="http://www.w3.org/2000/svg" height="35" viewBox="0 96 960 960" width="48">
            <path d="M220 896q-24 0-42-18t-18-42V693h60v143h520V693h60v143q0 24-18 42t-42 18H220Zm260-153L287 550l43-43 120 120V256h60v371l120-120 35 43-193 193Z"/>
          </svg>
        </div>
      </div>
    </div>
    <VFlexPagination
        v-if="filters.last_page > 1"
        :total-items="filters.total"
        :item-per-page="filters.per_page"
        :current-page="+filters.current_page"
        :max-links-displayed="7"
        @update:page="page.current = $event"
    />
  </div>
</template>

<style lang="scss" scoped>
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
}

.search_block {
  margin-left: 35px;
  height: 80px;
}
.search_input {
  display: flex;
  flex-direction: row;
  width: 330px;
  border-radius: 8px;
  border: 1px solid transparent;
  background-color: #1a1a1a;
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
.block {
  margin-bottom: 4px;
  display: flex;
  justify-content: center;
  flex-direction: row;
  width: 80%;
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