<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search & Results -->
    <div class="z-20 flex justify-center relative px-4 pt-8 pb-32 bg-hero-pattern bg-cover">

      <!-- search input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-2xl pb-4">IP Address Tracker</h1>

        <div class="flex">
          <span class="cursor-pointer bg-black text-white px-4 rounded-tl-md rounded-bl-md flex items-center" @click="getCurrentIP">
            <i class="fa-solid fa-circle-notch fa-spin" v-if="currentLocationLoading"></i>
            <i class="fas fa-map-marker-alt" v-else></i>
          </span>
          <input v-model="queryIP" type="text" class="flex-1 py-3 px-3 focus:outline-none" placeholder="Search for any IP address" @keyup.enter="getIPInfo">

          <span class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center" @click="getIPInfo">
            <i class="fa-solid fa-circle-notch fa-spin" v-if="loading"></i>
            <i class="fas fa-chevron-right" v-else></i>
          </span>
        </div>
      </div>

      <!-- IP Info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>

    <!-- map -->
     <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from '@/components/IPInfo'
import leaflet from 'leaflet'
import { onMounted, ref } from 'vue'
import axios from 'axios'

export default {
  name: 'HomeView',
  components: {
    IPInfo
  },
  setup() {
    let newMap;
    const queryIP = ref('')
    const ipInfo = ref(null)
    const apiKey = process.env.VUE_APP_MAPBOX_ACCESS_TOKEN
    const geoIPKey = process.env.VUE_APP_GEO_IP
    var currentLocationLoading = false
    var loading = false

    onMounted(() => {
      newMap = leaflet.map('map', { zoomControl: false }).setView([27.7172, 85.3240], 13)

      leaflet.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${apiKey}`, {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1,
          accessToken: `${apiKey}`
      }).addTo(newMap);
    })

    const getIPInfo = async () => {
      loading = true
      try {
        const response = await axios.get(`https://geo.ipify.org/api/v1?apiKey=${geoIPKey}&ipAddress=${queryIP.value}`)
        const result = response.data
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        }
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(newMap)
        newMap.setView([ipInfo.value.lat, ipInfo.value.lng], 16)
        loading = false
      } catch(error) {
        alert(error.message)
        loading = false
      }
    }

    const getCurrentIP = async () => {
      currentLocationLoading = true
      try {
        const response = await axios.get(`https://api.ipify.org/?format=json`)
        const currentIP = response.data.ip
        queryIP.value = currentIP
        getIPInfo()
        currentLocationLoading = false
      } catch(error) {
        alert(error.message)
        currentLocationLoading = false
      }
    }

    return {
      queryIP,
      ipInfo,
      getIPInfo,
      apiKey,
      geoIPKey,
      getCurrentIP,
      currentLocationLoading,
      loading
    }
  }
}
</script>
