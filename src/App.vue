<template>
  <div class="weather-dashboard p-6 bg-slate-50 min-h-screen">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold text-slate-800 tracking-tight">Weather Station</h1>
      <div v-if="weatherData.length > 0" class="text-right bg-white p-3 rounded-xl shadow-sm border border-slate-200">
        <p class="text-[10px] uppercase text-slate-400 font-bold tracking-widest">Last Update</p>
        <p class="text-sm font-semibold text-slate-700">{{ formatTime(weatherData[0].dateutc) }}</p>
      </div>
    </div>

    <div v-if="weatherData.length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      
      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Temperature</p>
        <p class="text-4xl font-bold text-slate-800 mt-2">{{ weatherData[0].tempf }}°F</p>
        <p class="text-xs text-slate-400 mt-1">Feels like {{ weatherData[0].feelsLike }}°F</p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Humidity</p>
        <p class="text-4xl font-bold text-blue-600 mt-2">{{ weatherData[0].humidity }}%</p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Wind Speed</p>
        <p class="text-4xl font-bold text-teal-500 mt-2">{{ weatherData[0].windspeedmph }} <span class="text-lg">mph</span></p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Pressure</p>
        <p class="text-4xl font-bold text-purple-600 mt-2">{{ weatherData[0].baromrelin }} <span class="text-lg text-slate-400">inHg</span></p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 col-span-1 md:col-span-2 lg:col-span-4">
        <div class="flex items-center gap-5">
          <div class="p-4 bg-yellow-100 rounded-2xl text-yellow-600 shadow-inner">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
            </svg>
          </div>
          <div>
            <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Last Lightning Strike</p>
            <p class="text-2xl font-bold text-slate-800">{{ weatherData[0].lightning_distance }} miles away</p>
            <p class="text-xs text-slate-400 mt-1 font-medium italic">
              Occurred on: {{ formatLightningTime(weatherData[0].lightning_time) }}
            </p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="weatherData.length > 0" class="grid grid-cols-1 md:grid-cols-3 gap-6 mt-8">
      <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
        <h3 class="text-xs font-bold text-slate-400 uppercase mb-4">Temperature Trend</h3>
        <div class="h-48"><Line :data="getChartData('tempf', '#ef4444')" :options="chartOptions" /></div>
      </div>
      <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
        <h3 class="text-xs font-bold text-slate-400 uppercase mb-4">Humidity Trend</h3>
        <div class="h-48"><Line :data="getChartData('humidity', '#2563eb')" :options="chartOptions" /></div>
      </div>
      <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
        <h3 class="text-xs font-bold text-slate-400 uppercase mb-4">Wind Trend</h3>
        <div class="h-48"><Line :data="getChartData('windspeedmph', '#14b8a6')" :options="chartOptions" /></div>
      </div>
    </div>

    <div v-else class="flex flex-col items-center justify-center mt-32">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500"></div>
      <p class="mt-4 text-slate-400 font-medium">Synchronizing with weather station...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { Line } from 'vue-chartjs';
import { Chart as ChartJS, CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend } from 'chart.js';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend);

const weatherData = ref([]);

const fetchWeatherData = async () => {
  try {
    const response = await fetch('/api/GetWeather');
    weatherData.value = await response.json();
  } catch (error) {
    console.error("API Sync Failed:", error);
  }
};

// Formats the chart datasets from the 20 historical records
const getChartData = (field, color) => {
  return {
    labels: weatherData.value.map(d => new Date(d.dateutc).toLocaleTimeString([], { hour: 'numeric', minute: '2-digit' })).reverse(),
    datasets: [{
      label: field,
      borderColor: color,
      backgroundColor: color,
      data: weatherData.value.map(d => d[field]).reverse(),
      tension: 0.3,
      pointRadius: 2
    }]
  };
};

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: { legend: { display: false } },
  scales: { x: { display: false }, y: { beginAtZero: false } }
};

const formatTime = (utc) => {
  return new Date(utc).toLocaleString('en-US', { hour: 'numeric', minute: '2-digit', second: '2-digit' });
};

const formatLightningTime = (ts) => {
  if (!ts || ts === 0) return "No recent history";
  return new Date(ts).toLocaleString('en-US', { 
    weekday: 'short', month: 'short', day: 'numeric', hour: 'numeric', minute: '2-digit' 
  });
};

onMounted(() => {
  fetchWeatherData();
  setInterval(fetchWeatherData, 300000); // 5-minute auto-refresh
});
</script>