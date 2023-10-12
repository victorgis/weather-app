<template>
  <div class="full-app">
    <Header />
    <div class="grid-container">
      <Loader v-if="isLoading" />

      <!-- First column  -->
      <div class="column1">
        <h1 class="name-location">{{ location ? location : 'Oron NG' }}</h1>
        <p class="name-weather">{{ weather ? weather : 'cloudy' }}</p>
        <span id="newIcon" class="cloud-size"><i class="fa-solid fa-cloud"></i></span>
        <p class="name-weather">{{ celsiusTemperature ? celsiusTemperature + ' °C' : '50.5°C' }}</p>
        <p class="">Feels like: {{ feelsLike ? feelsLike + ' °C' : '50.5°C' }}</p>
      </div>

      <!-- Second Column  -->
      <div class="column2">
        <div class="search-grid-container">
          <div class="date-time">
            <p>
              <b>
                <!-- <b>Today's Date & Time:</b>  -->
                {{ date ? date : 'Thu Oct 12 2023' }},
                {{ time ? time : 'Thu 14:21:40 GMT+0100' }}
              </b>
            </p>
            <!-- <p><b>Today's Time:</b> {{ time ? time : 'Thu 14:21:40 GMT+0100' }}</p> -->
          </div>
          <div class="search">
            <form @submit.prevent class="search-form">
              <input
                v-model="searchCity"
                type="text"
                id="search-input"
                placeholder="Search Location"
              />
              <button @click="handleSearchTerm()" type="submit">Search</button>
            </form>
          </div>
        </div>
        <div class="weather-today">
          <p class="name-weather">Today's Weather</p>
        </div>

        <!-- first boxes  -->
        <div class="factors-grid-container">
          <div class="box">
            <span><b>Pressure</b></span>
            <span>{{ pressure ? pressure + ' Pa' : '1000 Pa' }}</span>
          </div>
          <div class="box">
            <span><b>Humidity</b></span>
            <span>{{ humidity ? humidity + '%' : '30%' }}</span>
          </div>
          <div class="box">
            <span><b>Temperature</b></span>
            <span>{{ celsiusTemperature ? celsiusTemperature + ' °C' : '35.4 °C' }}</span>
          </div>
        </div>

        <!-- second boxes  -->
        <div class="factors-grid-container">
          <div class="box">
            <span><b>Visibility</b></span>
            <span>{{ visibility ? visibility / 1000 + ' km' : '5km' }}</span>
          </div>
          <div class="box">
            <span><b>Atm Condition</b></span>
            <span>{{ condition ? condition : 'Rain' }}</span>
          </div>
          <div class="box">
            <span><b>Wind Speed</b></span>
            <span>{{ windSpeed ? windSpeed + ' m/s' : '3.5 m/s' }}</span>
          </div>
        </div>

        <!-- third boxes  -->
        <div class="factors-grid-container">
          <div class="box">
            <span><b>Country Code</b></span>
            <span>{{ countryCode ? countryCode : 'NG' }}</span>
          </div>
          <div class="box">
            <span><b>Timezone</b></span>
            <span>{{ timeZoneOffset ? 'GMT' + timeZoneOffset : 'GMT +01:00' }}</span>
          </div>
          <div class="box">
            <span><b>Wind Deg</b></span>
            <span>{{ windDeg ? windDeg + '°' : '35°' }}</span>
          </div>
        </div>
      </div>
    </div>

    <Footer />
  </div>
</template>

<script>
import Header from './components/Header.vue'
import Footer from './components/Footer.vue'
import Loader from './components/Loader.vue'
import axios from 'axios'
export default {
  name: 'WeatherAppApp',
  components: {
    // eslint-disable-next-line vue/no-reserved-component-names
    Header,
    // eslint-disable-next-line vue/no-reserved-component-names
    Footer,
    // eslint-disable-next-line vue/no-unused-components
    Loader
  },
  data() {
    return {
      weatherIcon: '',
      weatherIconsList: ['<i class="fa-solid fa-cloud"></i>'],
      isLoading: false,
      latitude: '',
      longitude: '',
      weather: 'Cloudy',
      pressure: '',
      humidity: '',
      temperature: '',
      feelsLike: '',
      location: 'Oron NG',
      celsiusTemperature: '',
      searchCity: '',
      date: 'Thu Oct 12 2023',
      time: '',
      visibility: '',
      condition: '',
      windSpeed: '',
      windDeg: '',
      countryCode: '',
      timeZoneOffset: '',
      geoLat: '',
      geoLong: ''
    }
  },

  mounted() {
    this.getGeolocation()
    // this.loadWeather()
  },

  computed: {},

  methods: {
    getGeolocation() {
      const options = {
        enableHighAccuracy: true,
        timeout: 10000
      }
      const errorCallback = (error) => {
        console.log(error)
      }
      navigator.geolocation.getCurrentPosition(this.showPosition, errorCallback, options)
    },

    showPosition(position) {
      this.longitude = position.coords.longitude
      this.latitude = position.coords.latitude

      if (this.longitude) {
        this.loadWeather()
      }
    },

    async loadWeather() {
      function secondsToTimeZoneOffset(seconds) {
        const sign = seconds < 0 ? '-' : '+'
        const absSeconds = Math.abs(seconds)
        const hours = String(Math.floor(absSeconds / 3600)).padStart(2, '0')
        const minutes = String(Math.floor((absSeconds % 3600) / 60)).padStart(2, '0')
        return sign + hours + ':' + minutes
      }

      function getLocalTimeFormatted(timeZoneOffset) {
        const utcNow = new Date()

        const sign = timeZoneOffset.charAt(0) === '+' ? 1 : -1
        const hours = parseInt(timeZoneOffset.slice(1, 3))
        const minutes = parseInt(timeZoneOffset.slice(4))

        // Calculate the local time in the desired format
        const localTime = new Date(utcNow.getTime() + sign * ((hours - 1) * 60 + minutes) * 60000)

        // Format the local time according to the specified format
        const options = {
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit'
          // timeZoneName: 'short'
        }

        return localTime.toLocaleString('en-US', options) + ` GMT ${timeZoneOffset}`
      }

      this.isLoading = true
      const { data } = await axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${this.latitude}&lon=${this.longitude}&appid=ee914cdd33ab3acb4346421f7daf6fd4`
      )
      this.weather = data.weather[0].description
      this.humidity = data.main.humidity
      this.pressure = data.main.pressure
      this.temperature = data.main.temp
      this.feelsLike = (data.main.feels_like - 273.15).toFixed(2)
      this.celsiusTemperature = (this.temperature - 273.15).toFixed(2)
      this.humidity = data.main.humidity
      const cityCountry = `${data.name}, ${data.sys.country}`
      this.location = cityCountry
      this.visibility = data.visibility
      this.condition = data.weather[0].main
      this.windSpeed = data.wind.speed
      this.windDeg = data.wind.deg
      this.countryCode = data.sys.country

      //Weather icons
      const icon = document.getElementById('newIcon')
      if (this.weather == 'light rain') {
        icon.innerHTML = `<i class="fa-solid fa-cloud-rain"></i>`
      } else if (this.weather == 'moderate rain') {
        icon.innerHTML = `<i class="fa-solid fa-cloud-sun-rain"></i>`
      }

      // convert api timezone to real timezone
      const seconds = data.timezone
      this.timeZoneOffset = secondsToTimeZoneOffset(seconds)

      const now = new Date()
      const dateOptions = {
        weekday: 'short',
        month: 'short',
        day: 'numeric',
        year: 'numeric'
      }
      this.date = now.toLocaleString('en-US', dateOptions)
      // this.time = convertTimeToCustomFormat(this.timeZoneOffset)
      // const timeZoneOffset = '+01:00'
      const formattedTime = getLocalTimeFormatted(this.timeZoneOffset)
      console.log(formattedTime) // Example output: "19:08:34 GMT +0100"

      this.isLoading = false
    },

    async handleSearchTerm() {
      this.isLoading = true
      const { data } = await axios.get(
        `https://api.openweathermap.org/geo/1.0/direct?q=${this.searchCity}&limit=5&appid=ee914cdd33ab3acb4346421f7daf6fd4`
      )
      this.latitude = data[0].lat
      this.longitude = data[0].lon

      if (data) {
        this.loadWeather()
      }
      this.isLoading = false
    }
  },

  watch: {}
}
</script>

<style scoped>
div.full-app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  text-align: center;
  display: grid;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.grid-container {
  margin: 50px;
  display: grid;
  grid-template-columns: 1fr 1.8fr; /* This sets the column widths */
}

.column1 {
  background-color: #1d3557; /* Just for visualization */
  color: #f1faee;
  padding: 20px; /* Optional styling for the first column */
}

.column2 {
  background-color: #f1faee;
  color: #1d3557;
  padding: 20px;
}
.column1 span {
  display: block;
}

.name-location {
  font-size: 40px;
  font-weight: 800;
}
.name-weather {
  font-size: 30px;
}

.cloud-size {
  font-size: 200px;
}

.search-grid-container {
  display: grid;
  grid-template-columns: 1fr 1fr; /* This sets the column widths */
  border-bottom: #1d3557 1px solid;
  /* height: 300px; */
}

.search-form {
  margin: 16px 0px;
}

button {
  background: #1d3557;
  color: #f1faee;
}

.date-time {
  text-align: center;
}
.weather-today {
  border-bottom: #1d3557 1px solid;
}

.factors-grid-container {
  margin: 40px 0;
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* Create three equal-width columns */
  gap: 20px; /* Optional gap between boxes */
}

.box {
  background-color: #a8dadc; /* Just for visualization */
  padding: 20px; /* Optional styling for the boxes */
}
.box span {
  display: block;
}
</style>