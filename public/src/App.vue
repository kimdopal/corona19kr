<template>
  <div id="app">
    <GmapMap
      :center="center"
      :zoom="zoom"
      style="width: 100%; height: 100%"
      :options="{
        streetViewControl: false,
        mapTypeControl: false,
        fullscreenControl: false,
      }"
      @center_changed="updateCenter"
      @idle="sync"
    >
      <GmapMarker
        :key="1004"
        :position="center"
        :clickable="false"
        :draggable="false"
        :icon="{url: 'data:image/png;base64,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',
        scaledSize: {width: 40, height: 40}}"
      />
    </GmapMap>
    <button class="resync-btn" v-on:click="getCurrentPos">내 위치</button>
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
      center: {
        lat: 37.5533876,
        lng: 126.9706454,
      },
      lastMoved: {
        lat: 37.5533876,
        lng: 126.9706454
      },
      zoom: 14,
    }
  },
  methods: {
    updateCenter(event) {
      console.log('event', event.lat(), event.lng());
      this.lastMoved.lat = event.lat();
      this.lastMoved.lng = event.lng();
    },
    sync() {
      console.log('sync');
    },
    getCurrentPos(event) {
      const app = this;
      console.log('current pos', event);
      app.center.lat = 0;
      app.center.lng = 0;

      if (!("geolocation" in navigator)) {
        alert('Geolocation not supported on this browser.')
      } else {
        navigator.geolocation.getCurrentPosition(pos => {
          app.center.lat = Number(pos.coords.latitude);
          app.center.lng = Number(pos.coords.longitude);
          app.zoom = 14;
          console.log(`pos: ${app.center.lat}, ${app.center.lng}`);
        }, err => {
          alert(`Cannot get current location. ${err.message}`)
        })
      }
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

.resync-btn {
    position: fixed;
    z-index: 9999;
    left: calc(50% - 150px);
    width: 300px;
    height: 30px;
    bottom: 25%;
    background-color: white;
    box-shadow: 2px 2px 2px 2px #808080;
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
