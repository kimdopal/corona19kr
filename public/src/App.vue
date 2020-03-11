<template>
  <div id="app">
    <GmapMap
      :center="{lat:lat, lng:lng}"
      :zoom="zoom"
      style="width: 100%; height: 100%"
      :options="{
        streetViewControl: false,
        mapTypeControl: false,
        fullscreenControl: false,
      }"
    >
    </GmapMap>
  </div>
</template>

<script>
import Vue from 'vue'
import * as VueGoogleMaps from 'vue2-google-maps'

Vue.use(VueGoogleMaps, {
  load: {
    key: 'AIzaSyA8fpmwbu3MGmCAHVlYkUEQkDdtg6dS0Vc',
    libraries: 'places',
  }
})

export default {
  name: 'app',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      lat: 37.5533876,
      lng: 126.9706454,
      zoom: 14
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;
}

html {
  width: 100%;
  height: 100%;
}

body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}
</style>
