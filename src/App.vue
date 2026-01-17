<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import VueApexCharts from 'vue3-apexcharts';

// --- State ---
const weatherData = ref([]); 
const loading = ref(true);

async function fetchWeatherData() {
  try {
    const response = await fetch('/api/GetWeather');
    weatherData.value = await response.json();
  } catch (error) {
    console.error("API Error:", error);
  }
}

// --- Computed Stats ---
const current = computed(() => {
  if (!weatherData.value || !Array.isArray(weatherData.value) || weatherData.value.length === 0) return null;
  return weatherData.value[0]; 
});

// --- Date/Time Formatters (Central Time) ---
const formatTimeOnly = (timestamp) => {
  if (!timestamp) return '--';
  return new Date(timestamp).toLocaleTimeString('en-US', {
    timeZone: 'America/Chicago',
    hour: 'numeric',
    minute: '2-digit',
    hour12: true
  });
};

const formatDateOnly = (timestamp) => {
  if (!timestamp) return '--';
  return new Date(timestamp).toLocaleDateString('en-US', {
    timeZone: 'America/Chicago',
    weekday: 'short',
    month: 'short',
    day: 'numeric'
  });
};

const formatFullDateTime = (timestamp) => {
  if (!timestamp) return '--';
  return new Date(timestamp).toLocaleDateString('en-US', {
    timeZone: 'America/Chicago',
    month: 'short',
    day: 'numeric',
    hour: 'numeric',
    minute: '2-digit'
  });
};

// --- Chart Configuration ---
const commonChartOptions = computed(() => ({
  chart: {
    type: 'area',
    height: 300,
    fontFamily: 'inherit',
    zoom: { enabled: true, type: 'x', autoScaleYaxis: true },
    toolbar: { 
      show: true,
      tools: { download: false, selection: true, zoom: true, zoomin: true, zoomout: true, pan: true, reset: true }
    },
    animations: { enabled: false } 
  },
  dataLabels: { enabled: false },
  stroke: { curve: 'smooth', width: 2 },
  xaxis: { 
    type: 'datetime', 
    tooltip: { enabled: false },
    labels: { datetimeFormatter: { day: 'dd MMM', hour: 'HH:mm' } } 
  },
  tooltip: { x: { format: 'dd MMM h:mm tt' }, theme: 'light' },
  grid: { borderColor: '#f3f4f6' },
  fill: {
    type: 'gradient',
    gradient: { shadeIntensity: 1, opacityFrom: 0.4, opacityTo: 0.1, stops: [0, 100] }
  }
}));

// --- Series Data ---
const tempSeries = computed(() => [{ name: 'Temp (°F)', data: (weatherData.value || []).map(row => [row.dateutc, row.tempf]) }]);
const windSeries = computed(() => [{ name: 'Wind (mph)', data: (weatherData.value || []).map(row => [row.dateutc, row.windspeedmph]) }]);
const humiditySeries = computed(() => [{ name: 'Humidity (%)', data: (weatherData.value || []).map(row => [row.dateutc, row.humidity]) }]);
const pressureSeries = computed(() => [{ name: 'Pressure (inHg)', data: (weatherData.value || []).map(row => [row.dateutc, row.baromrelin]) }]);

// --- Fetch Data ---
const fetchData = async () => {
  try {
    const response = await axios.get('/data-api/rest/Weather?$orderby=dateutc desc&$top=2000'); 
    
    if (response.data && Array.isArray(response.data.value)) {
        weatherData.value = response.data.value;
    } else {
        throw new Error("Invalid API format");
    }
  } catch (err) {
    console.warn("API unavailable. Generating sample data...");
    loadSampleData();
  } finally {
    loading.value = false;
  }
};

const loadSampleData = () => {
  const data = [];
  const now = Date.now();
  const points = 336; // 7 days history
  
  for (let i = 0; i < points; i++) {
    const time = now - (i * 30 * 60 * 1000);
    const temp = 65 + Math.sin(i / 10) * 15 + Math.random() * 2;
    const wind = Math.abs(Math.cos(i / 5) * 10 + Math.random() * 5);
    const pressure = 29.9 + Math.sin(i / 20) * 0.4;
    
    data.push({
      dateutc: time,
      tempf: parseFloat(temp.toFixed(1)),
      windspeedmph: parseFloat(wind.toFixed(1)),
      humidity: Math.floor(60 + Math.random() * 30),
      baromrelin: parseFloat(pressure.toFixed(2)),
      dailyrainin: i < 5 ? 0.46 : 0,    // Rain today
      monthlyrainin: 2.35,              // Rain this month
      lightning_time: now - (1000 * 60 * 45), 
      lightning_distance: 4.2
    });
  }
  weatherData.value = data;
};

onMounted(() => {
fetchWeatherData();
});
</script>

<template>
  <div class="min-h-screen bg-slate-50 p-6 font-sans text-slate-800 flex flex-col">
    
    <header class="mb-8 max-w-7xl mx-auto w-full flex justify-between items-end border-b border-slate-200 pb-4">
      <div>
        <h1 class="text-3xl font-bold text-slate-900 tracking-tight">Weather Station</h1>
        <p class="text-slate-500 text-sm mt-1">Live Conditions</p>
      </div>
      <div v-if="current" class="text-right">
        <div class="bg-white px-4 py-2 rounded-lg shadow-sm border border-slate-100">
          <p class="text-xs text-slate-400 uppercase font-bold tracking-wider mb-1">Last Updated</p>
          <div class="leading-tight">
            <p class="text-sm font-semibold text-slate-500 uppercase tracking-wide">{{ formatDateOnly(current.dateutc) }}</p>
            <p class="text-xl font-mono font-bold text-slate-700">
              {{ formatTimeOnly(current.dateutc) }} <span class="text-sm text-slate-400">CT</span>
            </p>
          </div>
        </div>
      </div>
    </header>

    <div v-if="loading" class="text-center py-20 text-slate-400">Loading dashboard...</div>

    <div v-else class="max-w-7xl mx-auto w-full space-y-6 flex-grow">
      
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4" v-if="current">
        
        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
          <p class="text-slate-400 text-xs uppercase font-bold tracking-wider">Temperature</p>
          <div class="flex items-baseline gap-1 mt-1">
            <span class="text-4xl font-bold text-slate-800">{{ current.tempf }}</span>
            <span class="text-lg text-slate-400">°F</span>
          </div>
        </div>

        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
          <p class="text-slate-400 text-xs uppercase font-bold tracking-wider">Humidity</p>
          <div class="flex items-baseline gap-1 mt-1">
            <span class="text-4xl font-bold text-blue-500">{{ current.humidity }}</span>
            <span class="text-lg text-slate-400">%</span>
          </div>
        </div>

        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
          <p class="text-slate-400 text-xs uppercase font-bold tracking-wider">Wind</p>
          <div class="flex items-baseline gap-1 mt-1">
            <span class="text-4xl font-bold text-teal-500">{{ current.windspeedmph }}</span>
            <span class="text-lg text-slate-400">mph</span>
          </div>
        </div>

        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
          <p class="text-slate-400 text-xs uppercase font-bold tracking-wider">Pressure</p>
          <div class="flex items-baseline gap-1 mt-1">
            <span class="text-4xl font-bold text-purple-500">{{ current.baromrelin }}</span>
            <span class="text-lg text-slate-400">inHg</span>
          </div>
        </div>

        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
          <p class="text-slate-400 text-xs uppercase font-bold tracking-wider">Rain (Last 24h)</p>
          <div class="flex items-baseline gap-1 mt-1">
            <span class="text-4xl font-bold text-indigo-500">{{ current.dailyrainin }}</span>
            <span class="text-lg text-slate-400">"</span>
          </div>
        </div>

        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
          <p class="text-slate-400 text-xs uppercase font-bold tracking-wider">Rain (Last 30d)</p>
          <div class="flex items-baseline gap-1 mt-1">
            <span class="text-4xl font-bold text-indigo-600">{{ current.monthlyrainin }}</span>
            <span class="text-lg text-slate-400">"</span>
          </div>
        </div>

        <div class="col-span-1 md:col-span-2 bg-slate-800 text-white p-5 rounded-xl shadow-sm border border-slate-700 flex flex-col justify-between relative overflow-hidden">
          <div class="absolute top-0 right-0 p-4 opacity-10">
             <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="w-16 h-16"><path stroke-linecap="round" stroke-linejoin="round" d="m3.75 13.5 10.5-11.25L12 10.5h8.25L9.75 21.75 12 13.5H3.75Z" /></svg>
          </div>
          <p class="text-yellow-400 text-xs uppercase font-bold tracking-wider">Last Strike</p>
          <div>
            <div class="text-2xl font-bold">{{ current.lightning_distance }} mi</div>
            <div class="text-xs text-slate-400 mt-1">{{ formatFullDateTime(current.lightning_time) }}</div>
          </div>
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <h3 class="font-bold text-lg mb-4 text-slate-700">Temperature History</h3>
          <VueApexCharts type="area" height="300" :options="{ ...commonChartOptions, colors: ['#f97316'] }" :series="tempSeries" />
        </div>

        <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <h3 class="font-bold text-lg mb-4 text-slate-700">Wind Speed</h3>
          <VueApexCharts type="area" height="300" :options="{ ...commonChartOptions, colors: ['#14b8a6'] }" :series="windSeries" />
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <h3 class="font-bold text-lg mb-4 text-slate-700">Humidity</h3>
          <VueApexCharts type="area" height="250" :options="{ ...commonChartOptions, colors: ['#3b82f6'], yaxis: { min: 0, max: 100 } }" :series="humiditySeries" />
        </div>

        <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <h3 class="font-bold text-lg mb-4 text-slate-700">Barometric Pressure</h3>
          <VueApexCharts type="area" height="250" :options="{ ...commonChartOptions, colors: ['#a855f7'], yaxis: { decimalsInFloat: 2 } }" :series="pressureSeries" />
        </div>
      </div>

    </div>

    <footer class="mt-12 py-6 text-center border-t border-slate-200">
      <p class="text-slate-400 text-sm">
        Questions? Please contact 
        <a href="mailto:aaron@giambattista.io" class="text-blue-500 hover:text-blue-600 hover:underline font-medium transition-colors">
          aaron@giambattista.io
        </a>
      </p>
    </footer>

  </div>
</template>