<template>
  <div class="[ text-center relative h-full w-full ]">
    <div class="[ absolute w-full h-full flex flex-col justify-center items-center bg-gray-800 text-4xl transition-all duration-300 text-white z-10 ]" :class="[ startingGame ? 'opacity-0 pointer-events-none' : 'opacity-100' ]">
      City Quiz
      <div @click="startGame" class="[ text-xl font-bold text-gray-300 cursor-pointer px-4 py-2 mt-2 transition-all duration-300 hover:bg-gray-900 hover:text-white ]">Play</div>
    </div>
    <div class="[ container mx-auto pt-10 relative ]">
      <div :class="[ amountOfCitis > 0 ? 'text-green-700' : '' ]" class="[ text-xl ]"><span>{{ amountOfCitis }}</span> cities placed</div>
      <div class="[ text-xl ]"><span :class="[ score < 1000 ? 'text-red-700' : '' ]">{{ score }}</span> kilometers left</div>
      <div class="[ text-xl ]">Select the location of <div class="[ text-2xl font-bold ]">{{ currentCity }}</div></div>
      <div class="[ mt-10 ]">
        <gmap-map
          ref="gMap"
          class="map"
          language="en"
          :options="{fullscreenControl: false, styles: mapStyle}"
          :zoom="6"
          :center="center"
          @click="getLatLng"
        >
        </gmap-map>
      </div>
      <div v-if="distance > 0">
        <span :class="[ distanceCheck ? 'text-green-600' : 'text-red-700' ]">{{ distanceCheck ? 'Correct' : 'Wrong' }}</span>  
        <div>distance between is {{ distance }}km</div>
      </div>
      <div class="[ absolute w-full h-full bg-white shadow-2xl flex flex-col justify-center items-center transition duration-300 text-2xl left-0 ] box" :class="[ finished ? 'opacity-100' : 'opacity-0 pointer-events-none' ]">
        finished
        <div> you found <span :class="[ amountOfCitis > 0 ? 'text-green-700' : 'text-red-700' ]">{{ amountOfCitis }}</span> cities</div>
        <div @click="start" class="[ text-xl font-bold text-gray-900 cursor-pointer px-4 py-2 mt-2 transition-all duration-300 hover:bg-gray-900 hover:text-white ]">Try Again?</div>
      </div>
    </div>
  </div>
</template>

<script>
import mapStyle from '~/static/map/mapStyle.json'
import { capitalCities } from '~/static/map/capitalCities.json'

export default {
  data() {
    return {
      startingGame: false,
      map: {},
      latLng: {},
      currentLatLng: {},
      currentCity: '',
      distance: '',
      score: 1500,
      finished: false,
      amountOfCitis: 0,
      currentLocation: {},
      markers: [],
      center: { lat: 47.3769, lng: 8.541 },
      pins: {
        selected: "data:image/png;base64,iVBORw0KGgo...",
        notSelected: "data:image/png;base64,iVBORw0KGgo..."
      },
      mapStyle: mapStyle,
      clusterStyle: [
        {
          url: "https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m1.png",
          width: 56,
          height: 56,
          textColor: "#fff"
        }
      ]
    }
  },
  mounted() {
    this.getCity()
  },
  computed: {
    distanceCheck() {
      return 50 >= this.distance
    }
  },
  methods: {
    startGame() {
      this.startingGame = true
    },
    getLatLng(e) {
      var lat =  e.latLng.lat()
      var lng = e.latLng.lng()
      this.latLng = { lat, lng } 

      this.setMarker()
      this.checkDistance()
      this.shotCurrentCity()
      setTimeout( () => {
        this.getCity()
      }, 2000)

      this.distanceCheck ? this.amountOfCitis += 1 : ''
      if (this.score - this.distance < 0) {
        this.score = 0
      } else this.score -= this.distance
    },
    setMarker() {
      this.resetPins()
      var marker = new google.maps.Marker({
          position: this.latLng,
          map: this.$refs.gMap.$mapObject
      });
      this.markers.push(marker);
    },
    checkDistance() {
      var lat1 = this.latLng.lat
      var lon1 = this.latLng.lng
      var lat2 = this.currentLatLng.lat
      var lon2 = this.currentLatLng.lng
 
      if ((lat1 == lat2) && (lon1 == lon2)) {
        this.distance = 0;
      }
      else {
        var radlat1 = Math.PI * lat1/180;
        var radlat2 = Math.PI * lat2/180;
        var theta = lon1-lon2;
        var radtheta = Math.PI * theta/180;
        var dist = Math.sin(radlat1) * Math.sin(radlat2) + Math.cos(radlat1) * Math.cos(radlat2) * Math.cos(radtheta);
        if (dist > 1) {
          dist = 1;
        }
        dist = Math.acos(dist);
        dist = dist * 180/Math.PI;
        dist = dist * 60 * 1.1515;
        //km
        dist = dist * 1.609344 
        this.distance = Math.round(dist);
      }
    },
    shotCurrentCity() {
      var marker = new google.maps.Marker({
          position: this.currentLatLng,
          map: this.$refs.gMap.$mapObject
      });
      var infowindow = new google.maps.InfoWindow({
        content: `${this.currentCity} is here`
      });
      this.markers.push(marker);
      infowindow.open(this.$refs.gMap.$mapObject, marker);
    },
    getRandomNumber(max) {
      return Math.floor(Math.random() * Math.floor(max))
    },
    getCity() {
      var city = capitalCities[this.getRandomNumber(capitalCities.length - 1)]
      this.currentCity = city.capitalCity
      this.currentLatLng = { lat: Number(city.lat), lng: Number(city.long) }
    },
    resetPins() {
      for (let i = 0; i < this.markers.length; i++) {
        this.markers[i].setMap(null);
      }
    },
    finish() {
      this.finished = true
      this.resetPins()
      this.amountOfCitis = 0
      this.score = 1500
      this.distance = 0
    },
    start() {
      this.finished = false
    }
  },
  watch: {
    score() {
      if(this.score === 0) {
        this.finish()
      } 
    }
  }
}
</script>

<style lang="scss">
.map {
  width: 100%;
  height: 400px;
}
.box {
  top: 5%;
}
</style>
