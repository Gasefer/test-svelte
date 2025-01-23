<script lang="ts" setup>
import { ref, watch } from 'vue'

// Інтерфейс для типізації даних погоди
interface WeatherData {
  name: string
  main: {
    temp: number
  }
  weather: {
    description: string
  }[]
}

const API_KEY = 'e5aca9055b94aef4964cd9e9b71f7890'
const cityArray = [
  'Kyiv',
  'Lviv',
  'Odesa',
  'Kharkiv',
  'Dnipro',
  'Zaporizhzhia',
  'Lutsk',
  'Ivano-Frankivsk',
]
const city = ref(localStorage.getItem('selectedCity') || 'Kyiv')
const weather = ref<WeatherData | null>(null) // Визначення типу для weather
const history = ref<string[]>(JSON.parse(localStorage.getItem('cityHistory') || '[]'))

const fetchWeather = async () => {
  const response = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${API_KEY}&units=metric`,
  )
  const data = await response.json()
  weather.value = data

  const index = history.value.indexOf(city.value)
  if (index !== -1) {
    history.value.splice(index, 1)
  }
  history.value.unshift(city.value)
  localStorage.setItem('cityHistory', JSON.stringify(history.value))
}

const clearHistory = () => {
  if (history.value.length > 1) {
    history.value = [history.value[0]]
  }
  if (typeof window !== 'undefined' && window.localStorage) {
    localStorage.setItem('cityHistory', JSON.stringify(history.value))
  }
}

watch(city, (newCity) => {
  localStorage.setItem('selectedCity', newCity)
  fetchWeather()
})

fetchWeather()
</script>

<template>
  <div class="weather-container">
    <h1 class="weather-title">Погода</h1>
    <div class="weather-select">
      <label for="city-select" class="select-label">Оберіть місто:</label>
      <select id="city-select" v-model="city" class="select-dropdown">
        <option v-for="cityItem in cityArray" :key="cityItem" :value="cityItem">
          {{ cityItem }}
        </option>
      </select>
    </div>
    <div v-if="weather" class="weather-info">
      <h2 class="weather-city">{{ weather.name }}</h2>
      <p class="weather-temp">Температура: {{ weather.main.temp }} °C</p>
      <p class="weather-desc">Опис: {{ weather.weather[0].description }}</p>
    </div>
    <div class="history-section">
      <h3 class="history-title">Історія вибору міст:</h3>
      <ul class="history-list">
        <li v-for="(city, index) in history" :key="index" class="history-item">
          {{ city }}
        </li>
      </ul>
      <button @click="clearHistory" class="clear-history-btn">Очистити історію</button>
    </div>
  </div>
</template>

<style lang="scss">
#app {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  color: #b6b6b6;
  height: 100lvh;
}
.weather {
  &-container {
    padding: 20px;
    max-width: 600px;
    margin: 0 auto;
    font-family: Arial, sans-serif;
    margin: 100px 0px auto 0px;
  }
  &-title {
    text-align: center;
    margin-bottom: 20px;
  }
  &-select {
    margin-bottom: 20px;
  }
  &-label {
    font-weight: bold;
    margin-right: 10px;
  }
  &-dropdown {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  &-info {
    margin-bottom: 20px;
  }
  &-city {
    font-size: 1.5rem;
    color: #007bff;
    margin-bottom: 10px;
  }

  &-temp,
  &-desc {
    font-size: 1.2rem;
  }
}

.history {
  &-title {
    font-size: 1.3rem;
    margin-bottom: 10px;
  }

  &-list {
    list-style-type: none;
    padding: 0;
  }
  &-item {
    padding: 5px 0;
    border-bottom: 1px solid #eee;
    font-size: 1rem;
  }
}
.clear-history-btn {
  margin-top: 10px;
  padding: 8px 12px;
  background-color: #dc3545;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background-color: #c82333;
  }
}
</style>
