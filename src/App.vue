<template>
  <div class="container mx-auto">
    <h1>www</h1>
    <!-- <div id="map" class="h-40"></div> -->
  </div>
</template>


<script lang="ts">
import L from "leaflet"
import { ref, onMounted } from 'vue'

export default {

  setup() {
    let mymap = ref(null)
    function initMap() {
      mymap = L.map('map').setView([51.505, -0.09], 13);
      L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: 'mapbox/streets-v11',
        tileSize: 512,
        zoomOffset: -1,
        accessToken: 'pk.eyJ1Ijoid2VuZHl3dTAwNyIsImEiOiJja2l5bGluZTgwMHlmMnNtZ21qaHoyeWlyIn0.Udp2rvV6oUNILB_R4iObug'
      }).addTo(mymap);
    }
    onMounted(() => {
      initMap()
    })

    return {
      initMap
    }
  }
}
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  /* margin-top: 60px; */

}
</style>
