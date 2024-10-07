<template>
  <v-app id="app">
    <v-container fluid fill-height class="con">
      <v-row justify="center" align="center">
        <v-col cols="12" md="8">
          <v-card class="pa-4 elevation-5" id="con">
            <v-card-title class="text-center">
              <h2 class="text-h5 font-weight-bold">Weather App</h2>
            </v-card-title>
            <v-card-text>
              <!-- Text Field to Input Cities -->
              <v-text-field
                v-model="citiesInput"
                label="Enter cities (comma-separated)"
                @keyup.enter="fetchWeather"
                outlined
                color="primary"
              ></v-text-field>

              <!-- Button to Get Weather Data -->
              <v-btn color="primary" block @click="fetchWeather">
                Get Weather
              </v-btn>

              <v-spacer class="mb-2"></v-spacer>

              <!-- Error Alert -->
              <v-alert v-if="error" type="error" dense>{{ error }}</v-alert>

              <!-- Weather Carousel -->
              <v-carousel
                v-model="activeSlide"
                hide-delimiters
                class="weather-carousel"
                v-if="weatherData.length"
              >
                <v-carousel-item
                  v-for="(weather, index) in weatherData"
                  :key="index"
                  :value="index"
                >
                  <v-card class="mb-4 weather-card elevation-3">
                    <v-card-title class="text-center weather-city-title">
                      <h3 class="text-h6 font-weight-bold">
                        {{ weather.name }}
                        <v-btn icon small @click="removeCity(index)" class="ml-2">
                          <v-icon color="red darken-1">mdi-close-circle</v-icon>
                        </v-btn>
                      </h3>
                    </v-card-title>

                    <v-card-text>
                      <v-list-item-subtitle class="text-center text-capitalize weather-description">
                        <strong>{{ weather.weather[0].description }}</strong>
                      </v-list-item-subtitle>

                      <!-- Weather Details -->
                      <v-divider class="my-5"></v-divider>

                      <v-row dense class="text-center">
                        <!-- Temperature -->
                        <v-col cols="6">
                          <v-icon large color="blue">mdi-temperature-celsius</v-icon>
                          <div><strong>Temperature:</strong></div>
                          <div class="weather-value">{{ weather.main.temp }} °C</div>
                        </v-col>

                        <!-- Humidity -->
                        <v-col cols="6">
                          <v-icon large color="indigo">mdi-water-percent</v-icon>
                          <div><strong>Humidity:</strong></div>
                          <div class="weather-value">{{ weather.main.humidity }}%</div>
                        </v-col>

                        <!-- Wind Speed -->
                        <v-col cols="6">
                          <v-icon large color="green">mdi-weather-windy</v-icon>
                          <div><strong>Wind Speed:</strong></div>
                          <div class="weather-value">{{ weather.wind.speed }} m/s</div>
                        </v-col>

                        <!-- Pressure -->
                        <v-col cols="6">
                          <v-icon large color="orange">mdi-gauge</v-icon>
                          <div><strong>Pressure:</strong></div>
                          <div class="weather-value">{{ weather.main.pressure }} hPa</div>
                        </v-col>

                        <!-- Sunrise -->
                        <v-col cols="6">
                          <v-icon large color="yellow">mdi-weather-sunset-down</v-icon>
                          <div><strong>Sunrise:</strong></div>
                          <div class="weather-value">{{ formatTime(weather.sys.sunrise) }}</div>
                        </v-col>

                        <!-- Sunset -->
                        <v-col cols="6">
                          <v-icon large color="red darken-1">mdi-weather-sunset</v-icon>
                          <div><strong>Sunset:</strong></div>
                          <div class="weather-value">{{ formatTime(weather.sys.sunset) }}</div>
                        </v-col>
                      </v-row>
                    </v-card-text>
                  </v-card>
                </v-carousel-item>
              </v-carousel>
              
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
      citiesInput: "",
      weatherData: [],
      error: "",
      apiKey: "76dead9f57f010cd8ec201d93ad0ac2e", // Replace with your OpenWeatherMap API key
      apiBaseUrl: "https://api.openweathermap.org/data/2.5/weather",
      activeSlide: 0,
    };
  },
  mounted() {
    this.loadWeatherDataFromLocalStorage();
  },
  methods: {
    async fetchWeather() {
      const cities = this.citiesInput.trim().split(",");
      this.error = "";
      for (const city of cities) {
        try {
          const response = await fetch(
            `${this.apiBaseUrl}?q=${city.trim()}&units=metric&appid=${this.apiKey}`
          );
          const data = await response.json();
          if (data.cod === 200) {
            // Check if the city is already in the weatherData array
            if (!this.weatherData.some((item) => item.name === data.name)) {
              this.weatherData.unshift(data); // Add new city to the beginning of the array
            } else {
              this.error += `City already exists: ${city.trim()}\n`;
            }
          } else {
            this.error += `City not found: ${city.trim()}\n`;
          }
        } catch (error) {
          this.error += `Unable to fetch weather data for ${city.trim()}\n`;
        }
      }
      this.citiesInput = ""; // Clear input after fetching
      this.saveWeatherDataToLocalStorage();
      this.activeSlide = 0; // Reset active slide
    },
    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    },
    saveWeatherDataToLocalStorage() {
      localStorage.setItem("weatherData", JSON.stringify(this.weatherData));
    },
    loadWeatherDataFromLocalStorage() {
      const storedWeatherData = localStorage.getItem("weatherData");
      if (storedWeatherData) {
        this.weatherData = JSON.parse(storedWeatherData);
      }
    },
    removeCity(index) {
      this.weatherData.splice(index, 1);
      this.saveWeatherDataToLocalStorage();
      if (this.activeSlide >= this.weatherData.length) {
        this.activeSlide = this.weatherData.length - 1; // Update active slide if necessary
      }
    },
  },
};
</script>

<style>
body {
  font-family: "Roboto", sans-serif;
  margin: 0; /* Remove default body margins */
}

#app {
  background-image: url("./background.png");
  background-size: cover;
  background-repeat: no-repeat;
  display: flex;
  flex-direction: column;
}

#con {
  background-color: transparent; /* Change background color */
  color: rgb(255, 254, 254); /* Text color */
  border-radius: 12px;
}

.weather-card {
  border-radius: 12px;
  background-color: #f7f9fc; /* Light background for weather card */
}

.weather-city-title {
  color: #3f51b5; /* Primary theme color */
}

.weather-description {
  font-size: 16px;
  color: #424242; /* Darker color for descriptions */
}

.weather-value {
  font-size: 14px;
  font-weight: 500;
  color: #212121; /* General weather data color */
}

.weather-carousel {
  /* Optional styling for the carousel */
  max-height: 400px;
}
</style>