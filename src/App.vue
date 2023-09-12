<template>
  <div id="app">
    <header>
      <h1>Weather Dashboard</h1>
    </header>
    <main>
      <settings @updateLocation="updateLocation"></settings>
      <form @submit.prevent="fetchWeatherData">
        <label for="location">Select Location:</label>
        <input type="text" id="location" v-model="selectedLocation" />
        <button type="submit">Update</button>
      </form>
      <section class="weather-info">
        <h2>Current Weather in {{ selectedLocation }}</h2>
        <p>Temperature: {{ temperature }}°C</p>
        <p>Humidity: {{ humidity }}%</p>
        <p>Wind Speed: {{ windSpeed }} km/h</p>
      </section>
      <div id="map"></div>
      <canvas id="chart"></canvas>
    </main>
    <footer>
      <p>Copyright © 2023, Weather Dashboard</p>
    </footer>
  </div>
</template>

<script>
/* global google */
import Chart from "chart.js/auto";
import Settings from "./components/Settings.vue"; // Import the Settings component

export default {
  components: {
    Settings, // Register the Settings component
  },
  data() {
    return {
      selectedLocation: "London",
      temperature: null,
      humidity: null,
      windSpeed: null,
      chart: null,
      map: null,
      lat: 51.5074,
      lon: -0.1278,
    };
  },
  watch: {
    selectedLocation: "fetchWeatherData",
  },
  async mounted() {
    await this.fetchWeatherData();
    this.initChart();
    this.initMap();
  },
  methods: {
    updateLocation(newLocation) {
      this.selectedLocation = newLocation;
    },
    async fetchWeatherData() {
      const apiKey = "0014bddcb1d4873f83d0ccbd96eb790c";
      try {
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.selectedLocation}&units=metric&appid=${apiKey}`
        );
        if (!response.ok) {
          throw new Error("Failed to fetch weather data");
        }
        const data = await response.json();
        this.temperature = data.main.temp;
        this.humidity = data.main.humidity;
        this.windSpeed = data.wind.speed;
        this.lat = data.coord.lat;
        this.lon = data.coord.lon;
        this.initChart();
        this.initMap();
      } catch (error) {
        console.error("Error fetching weather data:", error);
      }
    },
    initChart() {
      if (this.chart) {
        this.chart.destroy();
      }
      const ctx = document.getElementById("chart").getContext("2d");
      const data = {
        labels: ["Temperature", "Humidity", "Wind Speed"],
        datasets: [
          {
            label: "Weather Data",
            data: [this.temperature, this.humidity, this.windSpeed],
            borderColor: "rgba(75, 192, 192, 1)",
            borderWidth: 1,
          },
        ],
      };
      const options = {
        responsive: true,
        maintainAspectRatio: false,
      };
      this.chart = new Chart(ctx, {
        type: "bar",
        data: data,
        options: options,
      });
    },
    initMap() {
      if (!google) return;
      const mapOptions = {
        center: { lat: this.lat, lng: this.lon },
        zoom: 10,
      };
      this.map = new google.maps.Map(
        document.getElementById("map"),
        mapOptions
      );
      const marker = new google.maps.Marker({
        position: { lat: this.lat, lng: this.lon },
        map: this.map,
        title: "Weather Station",
      });
      const infoWindow = new google.maps.InfoWindow({
        content: `<h4>${this.selectedLocation}</h4><p>Temperature: ${this.temperature}°C</p><p>Humidity: ${this.humidity}%</p>`,
      });
      marker.addListener("click", () => {
        infoWindow.open(this.map, marker);
      });
    },
  },
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

header,
footer {
  background-color: #4caf50;
  color: white;
  text-align: center;
  padding: 1em;
}

.weather-info {
  padding: 2em;
  text-align: center;
  transition: all 0.3s ease;
}

@media (max-width: 768px) {
  .weather-info {
    padding: 1em;
  }
}

#map {
  height: 400px;
  width: 100%;
}

#chart {
  height: 400px;
  width: 100%;
}
</style>
