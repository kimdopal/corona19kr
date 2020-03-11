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
        :icon="{url: 'https://firebasestorage.googleapis.com/v0/b/for-ww3.appspot.com/o/test-lab%2Fgame%20user.png?alt=media&token=f274ea01-ce82-48fe-a6c3-2244fb5ad926',
        scaledSize: {width: 48, height: 48}}"
      />

      <GmapMarker
        :key="index"
        v-for="(m, index) in stores"
        :position="{lat: m.lat, lng: m.lng}"
        :clickable="false"
        :draggable="false"
        :icon="{url: 'https://image.flaticon.com/icons/png/512/37/37134.png',
        scaledSize: {width: 40, height: 40}}"
      />
    </GmapMap>
    <button class="resync-btn" v-on:click="getCurrentPos">내 위치</button>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
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
      distance: 2000,
      stores: []
    }
  },
  mounted() {
    console.log('mounted');
    this.getMaskGeo();
  },
  methods: {
    updateCenter(event) {
      console.log('event', event.lat(), event.lng());
      this.lastMoved.lat = event.lat();
      this.lastMoved.lng = event.lng();
    },
    sync() {
      console.log('sync');
      this.getMaskGeo();
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
          app.lastMoved.lat = Number(pos.coords.latitude);
          app.lastMoved.lng = Number(pos.coords.longitude);
          app.zoom = 14;
          console.log(`pos: ${app.center.lat}, ${app.center.lng}`);
          this.getMaskGeo();
        }, err => {
          alert(`Cannot get current location. ${err.message}`)
        })
      }
    },
    getMaskGeo() {
      const app = this;
      axios.get(`https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/storesByGeo/json?lat=${app.lastMoved.lat}&lng=${app.lastMoved.lng}&m=${app.distance}`)
      .then((response) => response.data)
      .then(data => {
        app.stores = data.stores
      })
      .catch(err => {
        console.log('err', err);
      })
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
