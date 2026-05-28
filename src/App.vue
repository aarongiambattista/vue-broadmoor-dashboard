<template>
  <div id="app">
    <div class="dashboard">
      <!-- Header -->
      <div class="header">
        <h1>Broadmoor Weather</h1>
        <div class="header-controls">
          <button class="reset-btn" @click="resetZoom">RESET ZOOM</button>
          <div class="last-sync">
            <div class="sync-label">LAST SYNC</div>
            <div class="sync-time">{{ lastSyncTime }}</div>
          </div>
        </div>
      </div>

      <!-- Primary Stats Row -->
      <div class="stats-row">
        <div class="stat-card">
          <div class="stat-label">TEMPERATURE</div>
          <div class="stat-value temp-color">{{ currentData.tempf }}°F</div>
          <div class="stat-sub temp-color">Feels like {{ currentData.feelsLike }}°F</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">HUMIDITY</div>
          <div class="stat-value humidity-color">{{ currentData.humidity }}%</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">WIND SPEED</div>
          <div class="stat-value wind-color">{{ currentData.windspeedmph }} mph</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">PRESSURE</div>
          <div class="stat-value pressure-color">{{ currentData.baromrelin }}<br>inHg</div>
        </div>
      </div>

      <!-- Dew Point & UV Index Row (NEW) -->
      <div class="stats-row two-col">
        <div class="stat-card">
          <div class="stat-label">DEW POINT</div>
          <div class="stat-value dewpoint-color">{{ currentData.dewPoint }}°F</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">UV INDEX</div>
          <div class="stat-value uv-color">{{ currentData.uv }}</div>
          <div class="stat-sub uv-color">{{ uvDescription }}</div>
        </div>
      </div>

      <!-- Rainfall Row (NEW) -->
      <div class="stats-row five-col">
        <div class="stat-card">
          <div class="stat-label">HOURLY RAIN</div>
          <div class="stat-value rain-color">{{ currentData.hourlyrainin }}"</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">EVENT RAIN</div>
          <div class="stat-value rain-color">{{ currentData.eventrainin }}"</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">WEEKLY RAIN</div>
          <div class="stat-value rain-color">{{ currentData.weeklyrainin }}"</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">MONTHLY RAIN</div>
          <div class="stat-value rain-color">{{ currentData.monthlyrainin }}"</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">YEARLY RAIN</div>
          <div class="stat-value rain-color">{{ currentData.yearlyrainin }}"</div>
        </div>
      </div>

      <!-- Lightning Strike -->
      <div class="lightning-card">
        <div class="lightning-icon">
          <span>⚡</span>
        </div>
        <div class="lightning-info">
          <div class="lightning-label">LAST LIGHTNING STRIKE</div>
          <div class="lightning-distance">{{ currentData.lightning_distance }} miles away</div>
          <div class="lightning-time">Detected: {{ lightningTimeFormatted }}</div>
        </div>
      </div>

      <!-- 30-Day High/Low Stats (NEW) -->
      <div class="stats-row">
        <div class="stat-card highlow-card">
          <div class="stat-label">TEMPERATURE <span class="range-label">30-DAY RANGE</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val temp-color">{{ stats30d.tempHigh }}°F</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val temp-low-color">{{ stats30d.tempLow }}°F</span>
            </div>
          </div>
        </div>
        <div class="stat-card highlow-card">
          <div class="stat-label">HUMIDITY <span class="range-label">30-DAY RANGE</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val humidity-color">{{ stats30d.humidityHigh }}%</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val humidity-low-color">{{ stats30d.humidityLow }}%</span>
            </div>
          </div>
        </div>
        <div class="stat-card highlow-card">
          <div class="stat-label">WIND SPEED <span class="range-label">30-DAY RANGE</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val wind-color">{{ stats30d.windHigh }} mph</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val wind-low-color">{{ stats30d.windLow }} mph</span>
            </div>
          </div>
        </div>
        <div class="stat-card highlow-card">
          <div class="stat-label">PRESSURE <span class="range-label">30-DAY RANGE</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val pressure-color">{{ stats30d.pressureHigh }} inHg</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val pressure-low-color">{{ stats30d.pressureLow }} inHg</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Trend Charts -->
      <div class="charts-row">
        <div class="chart-card">
          <div class="chart-label temp-color">TEMPERATURE TREND</div>
          <canvas ref="tempChart"></canvas>
        </div>
        <div class="chart-card">
          <div class="chart-label humidity-color">HUMIDITY TREND</div>
          <canvas ref="humidityChart"></canvas>
        </div>
        <div class="chart-card">
          <div class="chart-label wind-color">WIND SPEED TREND</div>
          <canvas ref="windChart"></canvas>
        </div>
      </div>

      <!-- All-Time High/Low Stats -->
      <div class="section-heading">ALL-TIME RECORDS</div>
      <div class="stats-row">
        <div class="stat-card highlow-card alltime-card">
          <div class="stat-label">TEMPERATURE <span class="range-label">ALL-TIME</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val temp-color">{{ statsAllTime.tempHigh }}°F</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val temp-low-color">{{ statsAllTime.tempLow }}°F</span>
            </div>
          </div>
        </div>
        <div class="stat-card highlow-card alltime-card">
          <div class="stat-label">HUMIDITY <span class="range-label">ALL-TIME</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val humidity-color">{{ statsAllTime.humidityHigh }}%</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val humidity-low-color">{{ statsAllTime.humidityLow }}%</span>
            </div>
          </div>
        </div>
        <div class="stat-card highlow-card alltime-card">
          <div class="stat-label">WIND SPEED <span class="range-label">ALL-TIME</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val wind-color">{{ statsAllTime.windHigh }} mph</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val wind-low-color">{{ statsAllTime.windLow }} mph</span>
            </div>
          </div>
        </div>
        <div class="stat-card highlow-card alltime-card">
          <div class="stat-label">PRESSURE <span class="range-label">ALL-TIME</span></div>
          <div class="highlow-row">
            <div class="highlow-item">
              <span class="highlow-arrow high">▲</span>
              <span class="highlow-val pressure-color">{{ statsAllTime.pressureHigh }} inHg</span>
            </div>
            <div class="highlow-item">
              <span class="highlow-arrow low">▼</span>
              <span class="highlow-val pressure-low-color">{{ statsAllTime.pressureLow }} inHg</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import 'chartjs-adapter-date-fns';
import zoomPlugin from 'chartjs-plugin-zoom';

Chart.register(zoomPlugin);

export default {
  name: 'App',
  data() {
    return {
      weatherData: [],
      currentData: {
        tempf: '--',
        humidity: '--',
        windspeedmph: '--',
        baromrelin: '--',
        feelsLike: '--',
        dewPoint: '--',
        uv: '--',
        hourlyrainin: '--',
        eventrainin: '--',
        weeklyrainin: '--',
        monthlyrainin: '--',
        yearlyrainin: '--',
        lightning_distance: '--',
        lightning_time: null,
        lightning_day: 0,
      },
      stats30d: {
        tempHigh: '--',
        tempLow: '--',
        humidityHigh: '--',
        humidityLow: '--',
        windHigh: '--',
        windLow: '--',
        pressureHigh: '--',
        pressureLow: '--',
      },
      statsAllTime: {
        tempHigh: '--',
        tempLow: '--',
        humidityHigh: '--',
        humidityLow: '--',
        windHigh: '--',
        windLow: '--',
        pressureHigh: '--',
        pressureLow: '--',
      },
      lastSyncTime: '--',
      tempChart: null,
      humidityChart: null,
      windChart: null,
    };
  },
  computed: {
    lightningTimeFormatted() {
      if (!this.currentData.lightning_time) return '--';
      const d = new Date(this.currentData.lightning_time);
      const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
      const day = days[d.getDay()];
      let hours = d.getHours();
      const minutes = d.getMinutes().toString().padStart(2, '0');
      const ampm = hours >= 12 ? 'PM' : 'AM';
      hours = hours % 12 || 12;
      return `${day} ${hours}:${minutes} ${ampm}`;
    },
    uvDescription() {
      const uv = this.currentData.uv;
      if (uv === '--') return '';
      if (uv <= 2) return 'Low';
      if (uv <= 5) return 'Moderate';
      if (uv <= 7) return 'High';
      if (uv <= 10) return 'Very High';
      return 'Extreme';
    },
  },
  methods: {
    async fetchData() {
      try {
        const [weatherRes, statsRes] = await Promise.all([
          fetch('/api/GetWeather'),
          fetch('/api/GetWeatherStats'),
        ]);

        const data = await weatherRes.json();
        const stats = await statsRes.json();

        this.weatherData = data;

        if (data.length > 0) {
          const latest = data[0];
          this.currentData = {
            tempf: latest.tempf,
            humidity: latest.humidity,
            windspeedmph: latest.windspeedmph,
            baromrelin: latest.baromrelin,
            feelsLike: latest.feelsLike,
            dewPoint: latest.dewPoint,
            uv: latest.uv,
            hourlyrainin: latest.hourlyrainin,
            eventrainin: latest.eventrainin,
            weeklyrainin: latest.weeklyrainin,
            monthlyrainin: latest.monthlyrainin,
            yearlyrainin: latest.yearlyrainin,
            lightning_distance: latest.lightning_distance,
            lightning_time: latest.lightning_time,
            lightning_day: latest.lightning_day,
          };

          // Format last sync time
          const syncDate = new Date(latest.LocalTimeCST || latest.date);
          let hours = syncDate.getHours();
          const minutes = syncDate.getMinutes().toString().padStart(2, '0');
          const ampm = hours >= 12 ? 'PM' : 'AM';
          hours = hours % 12 || 12;
          this.lastSyncTime = `${hours}:${minutes} ${ampm}`;
        }

        // Map API stats to the format the template expects
        if (stats.stats30d) {
          const s = stats.stats30d;
          this.stats30d = {
            tempHigh: s.temp_high,
            tempLow: s.temp_low,
            humidityHigh: s.humidity_high,
            humidityLow: s.humidity_low,
            windHigh: s.wind_high,
            windLow: s.wind_low,
            pressureHigh: s.pressure_high,
            pressureLow: s.pressure_low,
          };
        }

        if (stats.statsAllTime) {
          const s = stats.statsAllTime;
          this.statsAllTime = {
            tempHigh: s.temp_high,
            tempLow: s.temp_low,
            humidityHigh: s.humidity_high,
            humidityLow: s.humidity_low,
            windHigh: s.wind_high,
            windLow: s.wind_low,
            pressureHigh: s.pressure_high,
            pressureLow: s.pressure_low,
          };
        }

        this.renderCharts(data);
      } catch (err) {
        console.error('Failed to fetch weather data:', err);
      }
    },

    renderCharts(data) {
      const reversed = [...data].reverse();
      const labels = reversed.map((d) => new Date(d.LocalTimeCST || d.date));

      const chartOptions = (color, unit) => ({
        responsive: true,
        maintainAspectRatio: false,
        interaction: {
          mode: 'index',
          intersect: false,
        },
        plugins: {
          legend: { display: false },
          zoom: {
            zoom: {
              wheel: { enabled: true },
              pinch: { enabled: true },
              mode: 'x',
            },
            pan: {
              enabled: true,
              mode: 'x',
            },
          },
        },
        scales: {
          x: {
            type: 'time',
            time: {
              tooltipFormat: 'MMM d, h:mm a',
              displayFormats: {
                hour: 'h:mm a',
                day: 'MMM d',
              },
            },
            ticks: {
              maxRotation: 45,
              autoSkip: true,
              maxTicksLimit: 8,
              font: { size: 10 },
              color: '#8e99a4',
            },
            grid: { display: false },
          },
          y: {
            ticks: {
              font: { size: 10 },
              color: '#8e99a4',
            },
            grid: {
              color: 'rgba(0,0,0,0.05)',
            },
          },
        },
      });

      // Destroy existing charts
      if (this.tempChart) this.tempChart.destroy();
      if (this.humidityChart) this.humidityChart.destroy();
      if (this.windChart) this.windChart.destroy();

      // Temperature chart
      this.tempChart = new Chart(this.$refs.tempChart, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              data: reversed.map((d) => d.tempf),
              borderColor: '#f87171',
              backgroundColor: 'rgba(248,113,113,0.1)',
              borderWidth: 2,
              pointRadius: 2,
              pointBackgroundColor: '#f87171',
              tension: 0.3,
              fill: true,
            },
          ],
        },
        options: chartOptions('#f87171', '°F'),
      });

      // Humidity chart
      this.humidityChart = new Chart(this.$refs.humidityChart, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              data: reversed.map((d) => d.humidity),
              borderColor: '#3b82f6',
              backgroundColor: 'rgba(59,130,246,0.1)',
              borderWidth: 2,
              pointRadius: 2,
              pointBackgroundColor: '#3b82f6',
              tension: 0.3,
              fill: true,
            },
          ],
        },
        options: chartOptions('#3b82f6', '%'),
      });

      // Wind speed chart
      this.windChart = new Chart(this.$refs.windChart, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              data: reversed.map((d) => d.windspeedmph),
              borderColor: '#2dd4bf',
              backgroundColor: 'rgba(45,212,191,0.1)',
              borderWidth: 2,
              pointRadius: 2,
              pointBackgroundColor: '#2dd4bf',
              tension: 0.3,
              fill: true,
            },
          ],
        },
        options: chartOptions('#2dd4bf', ' mph'),
      });
    },

    resetZoom() {
      if (this.tempChart) this.tempChart.resetZoom();
      if (this.humidityChart) this.humidityChart.resetZoom();
      if (this.windChart) this.windChart.resetZoom();
    },
  },

  mounted() {
    this.fetchData();
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'DM Sans', sans-serif;
  background: #f0f2f5;
  color: #2d3748;
}

#app {
  min-height: 100vh;
  padding: 24px;
}

.dashboard {
  max-width: 1100px;
  margin: 0 auto;
}

/* Header */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.header h1 {
  font-size: 28px;
  font-weight: 700;
  color: #1a202c;
}

.header-controls {
  display: flex;
  align-items: center;
  gap: 12px;
}

.reset-btn {
  padding: 8px 18px;
  border: 2px solid #cbd5e0;
  border-radius: 8px;
  background: white;
  font-family: 'DM Sans', sans-serif;
  font-size: 13px;
  font-weight: 600;
  color: #4a5568;
  cursor: pointer;
  transition: all 0.15s;
}

.reset-btn:hover {
  border-color: #a0aec0;
  background: #f7fafc;
}

.last-sync {
  background: #edf2f7;
  border-radius: 8px;
  padding: 6px 14px;
  text-align: center;
}

.sync-label {
  font-size: 10px;
  font-weight: 600;
  color: #a0aec0;
  letter-spacing: 0.5px;
}

.sync-time {
  font-size: 14px;
  font-weight: 700;
  color: #2d3748;
}

/* Stats Row */
.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 16px;
}

.stats-row.two-col {
  grid-template-columns: repeat(2, 1fr);
}

.stats-row.five-col {
  grid-template-columns: repeat(5, 1fr);
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
}

.stat-label {
  font-size: 12px;
  font-weight: 600;
  color: #a0aec0;
  letter-spacing: 0.5px;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 32px;
  font-weight: 700;
  line-height: 1.1;
}

.stat-sub {
  font-size: 13px;
  margin-top: 4px;
}

/* Color Classes */
.temp-color {
  color: #1a202c;
}

.humidity-color {
  color: #8b5cf6;
}

.wind-color {
  color: #0ea5e9;
}

.pressure-color {
  color: #f97316;
}

.dewpoint-color {
  color: #06b6d4;
}

.uv-color {
  color: #eab308;
}

.rain-color {
  color: #3b82f6;
}

.temp-low-color {
  color: #60a5fa;
}

.humidity-low-color {
  color: #c4b5fd;
}

.wind-low-color {
  color: #7dd3fc;
}

.pressure-low-color {
  color: #fdba74;
}

/* Lightning Card */
.lightning-card {
  background: #fffbeb;
  border-radius: 12px;
  padding: 20px 24px;
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
}

.lightning-icon {
  width: 48px;
  height: 48px;
  background: #fef3c7;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  flex-shrink: 0;
}

.lightning-label {
  font-size: 12px;
  font-weight: 600;
  color: #a0aec0;
  letter-spacing: 0.5px;
}

.lightning-distance {
  font-size: 20px;
  font-weight: 700;
  color: #1a202c;
}

.lightning-time {
  font-size: 13px;
  color: #68d391;
  font-style: italic;
}

/* High/Low Cards */
.highlow-card {
  padding: 16px 20px;
}

.range-label {
  font-size: 10px;
  color: #cbd5e0;
  margin-left: 6px;
}

.highlow-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
}

.highlow-item {
  display: flex;
  align-items: center;
  gap: 6px;
}

.highlow-arrow {
  font-size: 14px;
}

.highlow-arrow.high {
  color: #ef4444;
}

.highlow-arrow.low {
  color: #3b82f6;
}

.highlow-val {
  font-size: 22px;
  font-weight: 700;
}

/* Charts Row */
.charts-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-top: 16px;
}

/* Section Heading */
.section-heading {
  font-size: 13px;
  font-weight: 700;
  color: #a0aec0;
  letter-spacing: 1px;
  margin-top: 24px;
  margin-bottom: 12px;
}

/* All-Time Cards */
.alltime-card {
  border-top: 3px solid #e2e8f0;
}

.chart-card {
  background: white;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
  height: 300px;
}

.chart-label {
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.5px;
  margin-bottom: 8px;
}

.chart-card canvas {
  width: 100% !important;
  height: calc(100% - 28px) !important;
}

/* Responsive */
@media (max-width: 900px) {
  .stats-row {
    grid-template-columns: repeat(2, 1fr);
  }

  .stats-row.five-col {
    grid-template-columns: repeat(3, 1fr);
  }

  .charts-row {
    grid-template-columns: 1fr;
  }

  .chart-card {
    height: 260px;
  }
}

@media (max-width: 600px) {
  #app {
    padding: 12px;
  }

  .header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .stats-row,
  .stats-row.two-col,
  .stats-row.five-col {
    grid-template-columns: repeat(2, 1fr);
  }

  .stat-value {
    font-size: 24px;
  }

  .highlow-val {
    font-size: 18px;
  }
}
</style>
