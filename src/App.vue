<template>
  <v-app :class="backgroundClass">
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="8">
          <v-card class="pa-4">
            <v-card-title>
              <h2 class="text-center">Weather App</h2>
            </v-card-title>
            <v-card-text>
              <v-text-field
                v-model="city"
                label="Enter city"
                @keyup.enter="addCityWeather"
              ></v-text-field>
              <v-btn color="primary" @click="addCityWeather" :disabled="loading">
                {{ loading ? "Loading..." : "Add City" }}
              </v-btn>
              <v-alert v-if="error" type="error" class="mt-3">
                {{ error }}
              </v-alert>
              <v-row v-if="weatherData.length" class="mt-4">
                <v-col v-for="weather in weatherData" :key="weather.id" cols="12" md="6" lg="4">
                  <v-card class="weather-card">
                    <v-card-title class="text-h5">{{ weather.name }}</v-card-title>
                    <v-card-subtitle>
                      {{ weather.weather[0].description }}
                    </v-card-subtitle>
                    <v-card-text>
                      <p><strong>Temperature:</strong> {{ weather.main.temp }} °C</p>
                      <p><strong>Humidity:</strong> {{ weather.main.humidity }}%</p>
                      <p><strong>Pressure:</strong> {{ weather.main.pressure }} hPa</p>
                      <p><strong>Wind Speed:</strong> {{ weather.wind.speed }} m/s</p>
                      <p>
                        <strong>Sunrise:</strong> {{ formatTime(weather.sys.sunrise) }}
                      </p>
                      <p>
                        <strong>Sunset:</strong> {{ formatTime(weather.sys.sunset) }}
                      </p>
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      city: "",
      weatherData: [],
      error: "",
      loading: false,
    };
  },
  computed: {
    backgroundClass() {
      if (!this.weatherData.length) return "default-background";
      const firstWeather = this.weatherData[0].weather[0].main.toLowerCase();
      if (firstWeather.includes("cloud")) return "cloudy-background";
      if (firstWeather.includes("rain")) return "rainy-background";
      if (firstWeather.includes("clear")) return "sunny-background";
      return "default-background";
    },
  },
  methods: {
    async addCityWeather() {
      if (this.city.trim() === "") {
        this.error = "Please enter a city name.";
        return;
      }
      this.error = "";
      this.loading = true;
      try {
        const apiKey = "11d1f25cc230e0264de3eba2a3f6e28d"; // Replace with your OpenWeatherMap API key
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${apiKey}`
        );
        const data = await response.json();
        if (data.cod === 200) {
          // Check if the city is already in the list
          if (!this.weatherData.some((weather) => weather.id === data.id)) {
            this.weatherData.push(data);
          } else {
            this.error = "City is already displayed.";
          }
        } else {
          this.error = data.message || "City not found.";
        }
      } catch {
        this.error = "Unable to fetch weather data.";
      } finally {
        this.loading = false;
        this.city = "";
      }
    },
    formatTime(unixTime) {
      const date = new Date(unixTime * 1000);
      return date.toLocaleTimeString();
    },
  },
};
</script>

<style>
body {
  font-family: "Roboto", sans-serif;
}

.default-background {
  background-color: #e3f2fd;
}

.sunny-background {
  background: linear-gradient(to bottom, #ffeb3b, #ff9800);
}

.cloudy-background {
  background: linear-gradient(to bottom, #b0bec5, #90a4ae);
}

.rainy-background {
  background: linear-gradient(to bottom, #4fc3f7, #0288d1);
}

.v-app {
  min-height: 100vh;
}

.weather-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
</style>
