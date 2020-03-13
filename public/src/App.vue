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
        :icon="{url: mode == 'otaku' ? 'https://firebasestorage.googleapis.com/v0/b/for-ww3.appspot.com/o/test-lab%2Fgame%20user.png?alt=media&token=f274ea01-ce82-48fe-a6c3-2244fb5ad926' : null,
        scaledSize: {width: 48, height: 48}}"
      />

      <GmapMarker
        :key="index"
        v-for="(m, index) in stores"
        :position="{lat: m.lat, lng: m.lng}"
        :clickable="true"
        :draggable="false"
        :icon="{url: getMarkUrl(m, markers),
        scaledSize: {width: 16, height: 16}}"
        @click="showShopDetail(m)"
      />

      <GmapInfoWindow
        @closeclick="storeInfoWindow.show=false" 
        :opened="storeInfoWindow.show"
        :position="storeInfoWindow.pos"
        :options="{
              pixelOffset: {
                width: 0,
                height: -8
              }
            }"
      >
      <pre>{{storeInfoWindow.message}}</pre>
      </GmapInfoWindow>
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
      mode: 'default',
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
      stores: [],
      markers: {
        empty: 'https://firebasestorage.googleapis.com/v0/b/corona19kr.appspot.com/o/markers%2Fempty.png?alt=media&token=5f8ddf78-0a91-4fcd-bee0-f362f906c17d',
        few: 'https://firebasestorage.googleapis.com/v0/b/corona19kr.appspot.com/o/markers%2Ffew.png?alt=media&token=e24a357d-d43d-4596-9ea5-665edf4fe6e5',
        plenty: 'https://firebasestorage.googleapis.com/v0/b/corona19kr.appspot.com/o/markers%2Fplenty.png?alt=media&token=91e96743-5c4f-4a2b-8ef2-9b580865008a',
        some: 'https://firebasestorage.googleapis.com/v0/b/corona19kr.appspot.com/o/markers%2Fsome.png?alt=media&token=b18f1bfd-4657-40bd-a69b-72446a5f016a'
      },
      storeInfoWindow: {
        show: false,
        pos: {
          lat: 37.5533876,
          lng: 126.9706454
        },
        message: '',
        remain: null
      },
      remain: {
        empty: '1개 이하',
        few: '2개 이상 ~ 30개 미만',
        plenty: '100개 이상',
        some: '30개 이상 ~ 100개 미만'
      }
    }
  },
  computed: {
    getMarkUrl: () => {
      return (store, markers) => {
        const remain_stat = store.remain_stat;
        if (Object.keys(markers).includes(remain_stat)) {
          return markers[remain_stat]
        }
        return markers.empty
      }
    }
  },
  mounted() {
    this.mode = new URL(location.href).searchParams.get('mode')
    this.center = {
      lat: Number(new URL(location.href).searchParams.get('lat')) || 37.5533876,
      lng: Number(new URL(location.href).searchParams.get('lng')) || 126.9706454
    }
    this.zoom = Number(new URL(location.href).searchParams.get('zoom')) || 14
    console.log(`mounted. current mode is ${this.mode}`);
    this.getMaskGeo();
  },
  methods: {
    showShopDetail(store) {
      this.storeInfoWindow.remain = store.remain_stat || 'empty';
      this.storeInfoWindow.show = true;
      this.storeInfoWindow.pos.lat = store.lat;
      this.storeInfoWindow.pos.lng = store.lng;
      this.storeInfoWindow.message = `${store.name}\n${store.addr}\n남은 마스크 양: ${this.remain[this.storeInfoWindow.remain]}\n데이터 갱신: ${store.stock_at}\n입고: ${store.stock_at}`
      console.log('store', store);
    },
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
