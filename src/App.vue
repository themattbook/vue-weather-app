<template>
  <main id="app" :class="setMode()">
    <div class="p-10 lg:p-0 flex-1 flex flex-col justify-center items-center">
      <div class="grid grid-cols-1 gap-1 mt-10 mx-auto">
        <div>
          <div>
            <h1 class="font-bold text-2xl">
              {{ weatherData[0].date }} will be
            </h1>
          </div>
          <div>
            <p class="font-bold text-7xl">
              <i :class="setIcon(weatherData[0].shortForecast)"></i>
              {{ weatherData[0].temp }}&deg; F
            </p>
          </div>
          <div class="mt-5">
            <p class="font-light tracking-tight text-xl max-w-lg">
              {{ weatherData[0].forecast }}
            </p>
          </div>
        </div>
        <div>
          <h1 class="font-bold text-2xl mt-10">Upcoming weather</h1>
        </div>
        <div
          class="flex items-center"
          v-for="data in weatherData.slice(1, 5)"
          :key="data.date"
        >
          <div
            class="
              text-center
              flex flex-col
              items-center
              justify-center
              opacity-70
            "
          >
            <i :class="setIcon(data.shortForecast)"></i>
            <h1 class="font-bold text-2xl">
              {{ convertDate(data.startTime) }}
            </h1>
            <p>{{ convertYear(data.startTime) }}</p>
          </div>
          <div class="ml-5 mt-3 max-w-md">
            <h1 class="font-bold text-3xl">
              {{ data.date }}
            </h1>
            <p class="mt-3 max-w-xl opacity-80 text-md tracking-tight">
              {{ data.forecast }}
            </p>
            <p class="text-sm opacity-90 mt-1">
              Winds {{ data.windDirection }} at
              {{ data.windSpeed }}
            </p>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  components: {},
  data() {
    return {
      latitude: "",
      longitude: "",
      weatherURI: "",
      weatherData: [],
    };
  },
  methods: {
    getLocation() {
      navigator.geolocation.getCurrentPosition(this.showLocation);
    },
    showLocation(position) {
      this.latitude = position.coords.latitude;
      this.longitude = position.coords.longitude;
      this.getWeather();
    },
    getWeather() {
      axios
        .get(
          `https://api.weather.gov/points/${this.latitude},${this.longitude}`
        )
        .then((response) => {
          this.weatherURI = response.data.properties.forecast;
          this.showWeather();
        })
        .catch((error) => {
          this.weatherURI = error;
        });
    },
    showWeather() {
      axios
        .get(this.weatherURI)
        .then((response) => {
          let res = response.data.properties;
          for (let i = 0; i < res.periods.length; i++) {
            this.weatherData.push({
              date: res.periods[i].name,
              temp: res.periods[i].temperature,
              forecast: res.periods[i].detailedForecast,
              icon: res.periods[i].icon,
              windSpeed: res.periods[i].windSpeed,
              windDirection: res.periods[i].windDirection,
              shortForecast: res.periods[i].shortForecast,
              startTime: res.periods[i].startTime,
            });
          }
        })
        .catch((error) => {
          this.weatherURI = error;
        });
    },
    convertDate(x) {
      const date = new Date(x);
      const iso = date.toISOString().slice(8, 10);
      return iso;
    },
    convertYear(x) {
      const date = new Date(x);
      const month = date.toLocaleString("en-US", { month: "short" });
      return month;
    },
    setIcon(x) {
      switch (x) {
        default:
          return "fa-solid fa-cloud-exclamation ";
        case "Mostly Sunny then Chance Showers And Thunderstorms":
          return "fa-solid fa-cloud-sun-rain ";
        case "Mostly Clear then Slight Chance Rain Showers":
          return "fa-solid fa-cloud-sun-rain ";
        case "Mostly Sunny":
          return "fa-solid fa-cloud-sun ";
        case "Sunny":
          return "fa-solid fa-sun";
        case "Clear":
          return "fa-solid fa-sun";
        case "Mostly Clear":
          return "fa-solid fa-sun";
        case "Slight Chance Rain Showers then Mostly Sunny":
          return "fa-solid fa-cloud-sun-rain";
        case "Slight Chance Showers And Thunderstorms then Mostly Clear":
          return "fa-solid fa-cloud-sun-rain";
        case "Chance Showers And Thunderstorms":
          return "fa-solid fa-cloud-bolt";
        case "Slight Chance Showers And Thunderstorms":
          return "fa-solid fa-cloud-bolt";
      }
    },
    setMode() {
      const hour = new Date();
      console.log(hour.getHours());
      if (hour.getHours() >= 16) {
        return "min-h-screen relative flex-col flex text-white bg-gray-900";
      } else {
        return "min-h-screen relative flex-col flex text-gray-900 bg-yellow-200";
      }
    },
  },
  mounted() {
    this.getLocation();
  },
};
</script>

<style>
</style>
