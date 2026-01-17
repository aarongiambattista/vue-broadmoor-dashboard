<template>
  <div class="weather-dashboard p-6 bg-slate-50 min-h-screen">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold text-slate-800 tracking-tight">Broadmoor Weather</h1>
      <div class="flex items-center gap-4">
        <button 
          @click="resetAllCharts"
          class="px-4 py-2 bg-white text-slate-600 text-xs font-bold uppercase rounded-lg shadow-sm border border-slate-200 hover:bg-slate-50 transition-colors"
        >
          Reset Zoom
        </button>
        <div v-if="weatherData.length > 0" class="text-right bg-white p-3 rounded-xl shadow-sm border border-slate-200">
          <p class="text-[10px] uppercase text-slate-400 font-bold tracking-widest">Last Sync</p>
          <p class="text-sm font-semibold text-slate-700">{{ formatTime(weatherData[0].dateutc) }}</p>
        </div>
      </div>
    </div>

    <div v-if="weatherData.length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Temperature</p>
        <p class="text-4xl font-bold text-slate-800 mt-2">{{ weatherData[0].tempf }}°F</p>
        <p class="text-xs text-slate-400 mt-1 italic">Feels like {{ weatherData[0].feelsLike }}°F</p>
      </div>
      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Humidity</p>
        <p class="text-4xl font-bold text-blue-600 mt-2">{{ weatherData[0].humidity }}%</p>
      </div>
      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Wind Speed</p>
        <p class="text-4xl font-bold text-teal-500 mt-2">{{ weatherData[0].windspeedmph }} mph</p>
      </div>
      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
        <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Pressure</p>
        <p class="text-4xl font-bold text-purple-600 mt-2">{{ weatherData[0].baromrelin }} inHg</p>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 col-span-1 md:col-span-2 lg:col-span-4">
        <div class="flex items-center gap-5">
          <div class="p-4 bg-yellow-100 rounded-2xl text-yellow-600 shadow-inner">⚡</div>
          <div>
            <p class="text-xs text-slate-500 font-bold uppercase tracking-wider">Last Lightning Strike</p>
            <p class="text-2xl font-bold text-slate-800">{{ weatherData[0].lightning_distance }} miles away</p>
            <p class="text-xs text-slate-400 mt-1 font-medium italic">
              {{ formatLightningTime(weatherData[0].lightning_time) }}
            </p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="weatherData.length > 0" class="mt-8 space-y-6">
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
          <h3 class="text-xs font-bold text-slate-400 uppercase mb-4">Temperature Trend</h3>
          <div class="h-64"><Line ref="tempChart" :data="getChartData('tempf', '#ef4444')" :options="chartOptions" /></div>
        </div>
        <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
          <h3 class="text-xs font-bold text-slate-400 uppercase mb-4">Humidity Trend</h3>
          <div class="h-64"><Line ref="humChart" :data="getChartData('humidity', '#2563eb')" :options="chartOptions" /></div>
        </div>
        <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100">
          <h3 class="text-xs font-bold text-slate-400 uppercase mb-4">Wind Speed Trend</h3>
          <div class="h-64"><Line ref="windChart" :data="getChartData('windspeedmph', '#14b8a6')" :options="chartOptions" /></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { Line } from 'vue-chartjs';
import zoomPlugin from 'chartjs-plugin-zoom';
import { Chart as ChartJS, CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend } from 'chart.js';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend, zoomPlugin);

const weatherData = ref([]);
const tempChart = ref(null);
const humChart = ref(null);
const windChart = ref(null);

const resetAllCharts = () => {
  // Accesses the chart instances via refs and calls the zoom plugin reset
  if (tempChart.value?.chart) tempChart.value.chart.resetZoom();
  if (humChart.value?.chart) humChart.value.chart.resetZoom();
  if (windChart.value?.chart) windChart.value.chart.resetZoom();
};

const fetchWeatherData = async () => {
  const response = await fetch('/api/GetWeather');
  weatherData.value = await response.json();
};

const getChartData = (field, color) => ({
  labels: weatherData.value.map(d => new Date(d.dateutc).toLocaleTimeString([], { hour: 'numeric', minute: '2-digit' })).reverse(),
  datasets: [{
    label: field,
    borderColor: color,
    backgroundColor: color + '33',
    data: weatherData.value.map(d => d[field]).reverse(),
    tension: 0.3,
    fill: true
  }]
});

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    zoom: {
      pan: { enabled: true, mode: 'x' },
      zoom: { wheel: { enabled: true }, pinch: { enabled: true }, mode: 'x' }
    },
    legend: { display: false }
  },
  scales: { x: { grid: { display: false } }, y: { beginAtZero: false } }
};

const formatTime = (utc) => new Date(utc).toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' });
const formatLightningTime = (ts) => ts && ts !== 0 ? `Detected: ${new Date(ts).toLocaleString('en-US', { weekday: 'short', hour: 'numeric', minute: '2-digit' })}` : "No recent strikes recorded";

onMounted(fetchWeatherData);
</script>