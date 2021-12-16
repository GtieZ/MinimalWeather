<template>
  <div class="container mt-5 text-white">
    <h3 v-show="error" class="text-danger">{{ error }}</h3>

    <div v-if="apiResponse">
      <h1>{{ apiResponse.name }}</h1>
      <h5 class="mb-2">{{ getDescription }}</h5>

      <h1 class="mb-2">
        <i :class="getIcon" style="font-size: 300%"></i>
      </h1>

      <h1 class="mt-3" style="font-size: 500%">{{ getTemperature }}</h1>
      <p class="mt-3 mb-4">
        (<i>sensación térmica:</i>
        <strong>{{ Math.round(apiResponse.main.feels_like) }}º</strong>)
      </p>

      <div class="mx-5 my-4"><hr /></div>

      <h5>
        Humedad: <strong>{{ apiResponse.main.humidity }}%</strong>
      </h5>

      <h5>
        Viento: <strong>{{ getWindDirection }}</strong>
      </h5>
      <h5>
        Velocidad: <strong>{{ getWindSpeed }} Km/h</strong>
      </h5>
      <p>
        <i>(rachas: {{ getWindGust }} Km/h)</i>
      </p>

      <h5 class="mt-2">
        Mínima: <strong>{{ Math.round(apiResponse.main.temp_min) }}º</strong>
      </h5>
      <h5>
        Máxima: <strong>{{ Math.round(apiResponse.main.temp_max) }}º</strong>
      </h5>

      <div class="mx-5 my-4"><hr /></div>
    </div>
  </div>
</template>

<script>
import jsonIcons from "@/data/icons.json";
export default {
  name: "Home",
  data() {
    return {
      coords: null,
      error: null,
      apiResponse: null,
      icons: jsonIcons,
    };
  },
  computed: {
    getDescription() {
      return this.apiResponse.weather[0].description;
    },
    getIcon() {
      if (this.apiResponse) {
        let prefix = "wi wi-";
        let code = this.apiResponse.weather[0].id;
        let icon = this.icons[code].icon;

        let today = new Date();
        let currentTimeHour = today.getHours();
        let iconHour = null;

        if (code == 800 || code == 951) {
          if (
            currentTimeHour > this.getSunrise() &&
            currentTimeHour <= this.getSunset()
          ) {
            iconHour = "day-";
          } else {
            iconHour = "night-";
            icon = "clear";
          }
        } else {
          iconHour = "";
        }
        //console.log(prefix+iconHour+icon+" mt-4")
        return prefix + iconHour + icon + " mt-4";
      }
      return null;
    },
    getTemperature() {
      let temperature = Math.round(this.apiResponse.main.temp);
      return temperature.toString() + "º C";
    },
    getWindDirection() {
      let angle = Math.floor(this.apiResponse.wind.deg / 22.5 + 0.5);
      let directions = [
        "Norte",
        "Norte-noreste",
        "Noreste",
        "Este-noreste",
        "Este",
        "Este-sureste",
        "Sureste",
        "Sur-sureste",
        "Sur",
        "Sur-suroeste",
        "Suroeste",
        "Oeste-suroeste",
        "Oeste",
        "Oeste-noroeste",
        "Noroeste",
        "Norte-noroeste",
      ];
      return directions[angle % 16];
    },
    getWindSpeed() {
      let windSpeed = this.apiResponse.wind.speed * 3.6;
      windSpeed = Math.round(windSpeed * 10) / 10;
      return windSpeed;
    },
    getWindGust() {
      let windGust = this.apiResponse.wind.gust * 3.6;
      windGust = Math.round(windGust * 10) / 10;
      return windGust;
    },
  },
  methods: {
    async setResponse() {
      this.apiResponse = await this.apiService();
    },
    gettingPosition(pos) {
      this.coords = pos.coords;

      this.setResponse();
    },
    onLocationError(err) {
      this.error = err.message;
      console.log(err.message);
    },
    getLocation() {
      if (navigator.geolocation) {
        let options = {
          enableHighAccurancy: true,
          timeout: 5000,
          maximumAge: 0,
        };

        if (this.error) {
          this.error = null;
        }

        navigator.geolocation.getCurrentPosition(
          this.gettingPosition,
          this.onLocationError,
          options
        );
      } else {
        this.error = "This device do not admit Geolocation!";
        console.log(this.error);
      }
    },
    getQuery() {
      let key = "d06d417d62bd88ac30896ba104b9bbf8";
      let lat = this.coords.latitude.toString();
      let lon = this.coords.longitude.toString();
      let header =
        "https://api.openweathermap.org/data/2.5/weather?units=metric&lang=es&appid=";

      let endpoint = header + key + "&lat=" + lat + "&lon=" + lon;
      return endpoint;
    },
    async getJSON(response) {
      if (response.status === 204) return "";
      return response.json();
    },
    apiService() {
      return fetch(this.getQuery())
        .then(this.getJSON)
        .catch((error) => (this.error = error));
    },
    getSunrise() {
      let sunriseUT = parseInt(this.apiResponse.sys.sunrise);
      let sunrise = new Date(sunriseUT * 1000);
      let hour = sunrise.getUTCHours();
      return hour;
    },
    getSunset() {
      let sunsetUT = parseInt(this.apiResponse.sys.sunset);
      let sunset = new Date(sunsetUT * 1000);
      let hour = sunset.getUTCHours();
      return hour;
    },
  },

  created() {
    this.getLocation();
  },
};
</script>
