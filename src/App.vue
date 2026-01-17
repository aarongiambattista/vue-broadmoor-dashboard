<template>
  <div class="weather-dashboard p-6 bg-slate-50 min-h-screen">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold text-slate-800">Weather Station</h1>
      <div v-if="weatherData.length > 0" class="text-right bg-white p-3 rounded-lg shadow-sm border border-slate-200">
        <p class="text-xs uppercase text-slate-500 font-bold">Last Updated</p>
        <p class="text-sm font-semibold text-slate-700">{{ formatTime(weatherData[0].dateutc) }}</p>
      </div>
    </div>

    <div v-if="weatherData.length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      
      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-sm text-slate-500 font-medium uppercase">Temperature</p>
        <p class="text-4xl font-bold text-slate-800 mt-2">{{ weatherData[0].tempf }}°F</p>
        <p class="text-xs text-slate-400 mt-1">Feels like {{ weatherData[0].feelsLike }}°F</p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-sm text-slate-500 font-medium uppercase">Humidity</p>
        <p class="text-4xl font-bold text-blue-600 mt-2">{{ weatherData[0].humidity }}%</p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-sm text-slate-500 font-medium uppercase">Wind Speed</p>
        <p class="text-4xl font-bold text-teal-500 mt-2">{{ weatherData[0].windspeedmph }} <span class="text-lg">mph</span></p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-sm text-slate-500 font-medium uppercase">Pressure</p>
        <p class="text-4xl font-bold text-purple-600 mt-2">{{ weatherData[0].baromrelin }} <span class="text-lg text-slate-400">inHg</span></p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 col-span-1 md:col-span-2">
        <div class="flex items-center gap-4">
          <div class="p-3 bg-yellow-100 rounded-full text-yellow-600">⚡</div>
          <div>
            <p class="text-sm text-slate-500 font-medium uppercase">Last Strike Distance</p>
            <p class="text-2xl font-bold text-slate-800">{{ weatherData[0].lightning_distance }} miles away</p>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="flex flex-col items-center justify-center mt-20">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500"></div>
      <p class="mt-4 text-slate-500 font-medium text-lg">Loading dashboard data...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const weatherData = ref([]);

const fetchWeatherData = async () => {
  try {
    const response = await fetch('/api/GetWeather');
    if (!response.ok) throw new Error("Network response was not ok");
    const data = await response.json();
    weatherData.value = data;
    console.log("Data loaded successfully:", data[0]);
  } catch (error) {
    console.error("Fetch error:", error);
  }
};

const formatTime = (utc) => {
  if (!utc) return "N/A";
  // Your API returns dateutc as a timestamp (ms)
  return new Date(utc).toLocaleString('en-US', { 
    weekday: 'short', 
    month: 'short', 
    day: 'numeric',
    hour: 'numeric', 
    minute: '2-digit',
    timeZoneName: 'short' 
  });
};

onMounted(() => {
  fetchWeatherData();
  // Optional: Refresh data every 5 minutes
  setInterval(fetchWeatherData, 300000);
});
</script>
