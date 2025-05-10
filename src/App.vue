<template>
  <div class="app-container">
    <div class="weather-app">
      <header class="app-header">
        <h1>WeatherCast</h1>
        <div class="search-container">
          <input v-model="city" placeholder="Search city..." @keyup.enter="getWeather" class="search-input" />
          <button @click="getWeather" class="search-button">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
              stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            </svg>
          </button>
        </div>
      </header>

      <div v-if="loading" class="loading-container">
        <div class="loading-spinner"></div>
        <p>Fetching weather data...</p>
      </div>

      <div v-if="error" class="error-message">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
          stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10"></circle>
          <line x1="12" y1="8" x2="12" y2="12"></line>
          <line x1="12" y1="16" x2="12.01" y2="16"></line>
        </svg>
        <p>{{ error }}</p>
      </div>

      <main v-if="weather" class="weather-content">
        <div class="current-weather">
          <div class="location-info">
            <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>
            <p class="current-date">{{ currentDate }}</p>
          </div>

          <div class="weather-main">
            <div class="temperature-display">
              <img :src="weatherIcon" :alt="weather.weather[0].description" class="weather-icon" />
              <div>
                <span class="current-temp">{{ Math.round(weather.main.temp) }}</span>
                <span class="temp-unit">°C</span>
              </div>
            </div>
            <p class="weather-description">{{ weather.weather[0].description }}</p>
          </div>

          <div class="weather-details">
            <div class="detail-item">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
                stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M17.7 7.7a2.5 2.5 0 1 1 1.8 4.3H2"></path>
                <path d="M9.6 4.6A2 2 0 1 1 11 8H2"></path>
                <path d="M12.6 19.4A2 2 0 1 0 14 16H2"></path>
              </svg>
              <span>{{ weather.wind.speed }} km/h</span>
            </div>
            <div class="detail-item">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
                stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M18 8h1a4 4 0 0 1 0 8h-1"></path>
                <path d="M12 8h1a4 4 0 0 1 0 8h-1"></path>
                <line x1="2" y1="8" x2="4" y2="8"></line>
                <line x1="4" y1="16" x2="2" y2="16"></line>
              </svg>
              <span>{{ weather.main.humidity }}%</span>
            </div>
            <div class="detail-item">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
                stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <line x1="4" y1="9" x2="20" y2="9"></line>
                <line x1="4" y1="15" x2="20" y2="15"></line>
                <line x1="10" y1="3" x2="8" y2="21"></line>
                <line x1="16" y1="3" x2="14" y2="21"></line>
              </svg>
              <span>{{ weather.main.pressure }} hPa</span>
            </div>
          </div>
        </div>

        <div v-if="forecast.length" class="forecast-section">
          <h3>5-Day Forecast</h3>
          <div class="forecast-days">
            <div v-for="(day, index) in forecast" :key="day.dt" class="forecast-day"
              :class="{ 'active-day': index === 0 }">
              <p class="day-name">{{ getDayName(day.dt_txt, index) }}</p>
              <img :src="getWeatherIcon(day.weather[0].icon)" :alt="day.weather[0].description" class="forecast-icon" />
              <div class="forecast-temps">
                <span class="high-temp">{{ Math.round(day.main.temp_max) }}°</span>
                <span class="low-temp">{{ Math.round(day.main.temp_min) }}°</span>
              </div>
            </div>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import { fetchWeather, fetchForecast } from './services/weatherService';

export default {
  setup() {
    const city = ref('');
    const weather = ref(null);
    const forecast = ref([]);
    const loading = ref(false);
    const error = ref(null);

    const currentDate = computed(() => {
      return new Date().toLocaleDateString('en-US', {
        weekday: 'long',
        month: 'short',
        day: 'numeric'
      });
    });

    const weatherIcon = computed(() => {
      if (!weather.value) return '';
      return `https://openweathermap.org/img/wn/${weather.value.weather[0].icon}@2x.png`;
    });

    const getDayName = (dateString, index) => {
      if (index === 0) return 'Today';
      const date = new Date(dateString);
      return date.toLocaleDateString('en-US', { weekday: 'short' });
    };

    const getWeatherIcon = (iconCode) => {
      return `https://openweathermap.org/img/wn/${iconCode}.png`;
    };

    const getWeather = async () => {
      if (!city.value) return;
      loading.value = true;
      error.value = null;

      try {
        weather.value = await fetchWeather(city.value);
        const forecastData = await fetchForecast(city.value);
        forecast.value = forecastData.list.filter((_, index) => index % 8 === 0).slice(0, 5);
      } catch (err) {
        error.value = err.message || 'Failed to fetch weather data';
      } finally {
        loading.value = false;
      }
    };

    return {
      city,
      weather,
      forecast,
      loading,
      error,
      currentDate,
      weatherIcon,
      getDayName,
      getWeatherIcon,
      getWeather
    };
  },
};
</script>

<style>
:root {
  --primary-color: #4361ee;
  --secondary-color: #3f37c9;
  --accent-color: #4895ef;
  --light-color: #f8f9fa;
  --dark-color: #212529;
  --text-color: #2b2d42;
  --text-light: #8d99ae;
  --success-color: #4cc9f0;
  --warning-color: #f72585;
  --border-radius: 12px;
  --box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
  --transition: all 0.3s ease;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  color: var(--text-color);
  min-height: 100vh;
  padding: 20px;
}

.app-container {
  max-width: 800px;
  margin: 0 auto;
}

.weather-app {
  background: rgba(255, 255, 255, 0.95);
  border-radius: var(--border-radius);
  box-shadow: var(--box-shadow);
  overflow: hidden;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.app-header {
  padding: 20px;
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  color: white;
  text-align: center;
}

.app-header h1 {
  font-size: 1.8rem;
  font-weight: 600;
  margin-bottom: 20px;
  letter-spacing: 0.5px;
}

.search-container {
  display: flex;
  max-width: 500px;
  margin: 0 auto;
  position: relative;
}

.search-input {
  flex: 1;
  padding: 12px 20px;
  border: none;
  border-radius: 50px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: var(--transition);
}

.search-input:focus {
  outline: none;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.search-button {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: transparent;
  border: none;
  color: var(--primary-color);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  gap: 20px;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(67, 97, 238, 0.2);
  border-radius: 50%;
  border-top-color: var(--primary-color);
  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.error-message {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 20px;
  background: rgba(247, 37, 133, 0.1);
  color: var(--warning-color);
  border-radius: var(--border-radius);
  margin: 20px;
}

.weather-content {
  padding: 20px;
}

.current-weather {
  margin-bottom: 30px;
}

.location-info {
  text-align: center;
  margin-bottom: 20px;
}

.location-info h2 {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 5px;
}

.current-date {
  color: var(--text-light);
  font-size: 0.9rem;
}

.weather-main {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 30px;
}

.temperature-display {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.weather-icon {
  width: 80px;
  height: 80px;
  margin-right: 15px;
}

.current-temp {
  font-size: 3.5rem;
  font-weight: 300;
  line-height: 1;
}

.temp-unit {
  font-size: 1.5rem;
  vertical-align: super;
  margin-left: 2px;
}

.weather-description {
  text-transform: capitalize;
  font-size: 1.1rem;
  color: var(--text-light);
}

.weather-details {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  background: rgba(248, 249, 250, 0.7);
  padding: 15px;
  border-radius: var(--border-radius);
}

.detail-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  font-size: 0.9rem;
}

.detail-item svg {
  color: var(--accent-color);
}

.forecast-section {
  margin-top: 30px;
}

.forecast-section h3 {
  font-size: 1.2rem;
  margin-bottom: 15px;
  color: var(--text-light);
  font-weight: 500;
}

.forecast-days {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 10px;
}

.forecast-day {
  background: rgba(248, 249, 250, 0.7);
  border-radius: var(--border-radius);
  padding: 15px 10px;
  text-align: center;
  transition: var(--transition);
}

.forecast-day.active-day {
  background: rgba(67, 97, 238, 0.1);
  border: 1px solid rgba(67, 97, 238, 0.2);
}

.forecast-day:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.day-name {
  font-weight: 500;
  margin-bottom: 10px;
}

.forecast-icon {
  width: 40px;
  height: 40px;
  margin: 5px auto;
}

.forecast-temps {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 5px;
}

.high-temp {
  font-weight: 600;
}

.low-temp {
  color: var(--text-light);
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .weather-app {
    border-radius: 0;
  }

  .app-header h1 {
    font-size: 1.5rem;
  }

  .current-temp {
    font-size: 3rem;
  }

  .weather-details {
    grid-template-columns: 1fr;
  }

  .forecast-days {
    grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
  }
}

@media (max-width: 480px) {
  body {
    padding: 10px;
  }

  .temperature-display {
    flex-direction: column;
    text-align: center;
  }

  .weather-icon {
    margin-right: 0;
    margin-bottom: 10px;
  }
}
</style>