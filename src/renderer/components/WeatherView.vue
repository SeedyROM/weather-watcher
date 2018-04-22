<template>
  <section>
    <transition name="fade-loader">
      <div v-if="!loaded" id="loading-spinner">
        <div>Loading...</div>
      </div>
    </transition>
    <transition name="fade-forecast">
      <div v-if="failedToFetch && !loaded">
        Failed to load weather!
      </div>
      <div v-cloak v-if="loaded" id="wrapper">
        <h1 id="city">
          <div>{{ weatherData.name }}</div>
          <div id="tempurature">{{ getTempurature() }}°<span>(F)</span></div>
        </h1>
        <h4 id="date">{{ formatTime("LLLL") }}</h4>
        <div id="condition">
          <div id="description">
            {{ getCondition() }}
            <div class="temp">High: {{ getMaxTempurature() }}°</div>            
            <div class="temp">Low: {{ getMinTempurature() }}°</div>
          </div>
          <i :class=getConditionImage()></i>
        </div>
        <div id="forecast" v-if="forecast">
          <!-- <div class="day" :key=index v-for="(day, index) in forecast.slice(8)">
            <span>{{ day.main }}</span>
          </div> -->
        </div>
      </div>
    </transition>
  </section>
</template>

<script>
  import moment from 'moment'
  import { setInterval } from 'timers'

  export default {
    name: 'weather-view',
    methods: {
      loadWeather () {
        this.$http
          .get('//api.openweathermap.org/data/2.5/weather?zip=97210,us&units=imperial&APPID=172e53545a1e41a124daaefa77c8a667')
          .then(({data}) => {
            console.dir(data)
            this.weatherData = data
            this.loaded = true
          })
          .catch(error => {
            this.failedToFetch = true
            console.dir(error)
          })
      },
      loadForecast () {
        this.$http
          .get('//api.openweathermap.org/data/2.5/forecast?zip=97210,us&units=imperial&APPID=172e53545a1e41a124daaefa77c8a667')
          .then(({data}) => {
            this.forecast = data.list
          })
      },
      formatTime (format) {
        return this.currentTime.format(format)
      },
      getCondition () {
        return this.weatherData.weather[0].main
      },
      getConditionImage () {
        return 'owf owf-' + this.weatherData.weather[0].id + this.iconSuffix()
      },
      getTempurature () {
        return this.weatherData.main.temp
      },
      getMinTempurature () {
        return this.weatherData.main.temp_max
      },
      getMaxTempurature () {
        return this.weatherData.main.temp_min
      },
      iconSuffix () {
        const now = moment()
        return (now > this.weatherData.sys.sunrise && now < this.weatherData.sys.sunset) ? '-d' : '-n'
      }
    },
    data: function () {
      return {
        loaded: false,
        failedToFetch: false,
        weatherData: null,
        forecast: null,
        currentTime: moment()
      }
    },
    created: function () {
      // Update the clock every second
      this.ticker = setInterval(() => {
        this.currentTime = moment()
      }, 1000)
      // Update the forecast every minute
      this.nextReload = setInterval(() => {
        this.loadWeather()
        this.loadForecast()
      }, 1000 * 60)
      // Load it up!
      this.loadWeather()
      this.loadForecast()
    },
    destroyed: function () {
      clearInterval(this.ticker)
    }
  }
</script>

<style lang="scss">
  @import '../styles/owfont-regular';
  @import './WeatherView/styles/main';
</style>