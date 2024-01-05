<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">No locations added. To start tracking a location, search in the field above.</p>
</template>

<script setup>
import axios from 'axios';
import CityCard from './City.Card.vue';
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const savedCities = ref([]);

const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'));

    const requests = [];
    const openWeatherApiKey = 'b059e5169ecc3624f8deb80eea598550';

    savedCities.value.forEach((city) => {
      requests.push(axios.get(`https://api.openweathermap.org/data/3.0/onecall?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=${openWeatherApiKey}&units=metric`));
    });

    const weatherData = await Promise.all(requests);

    // Flicker Delay
    await new Promise((res) => setTimeout(res, 1000));

    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value.data;
    });
  }
};

await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: {
      state: city.state,
      city: city.city,
    },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
};
</script>
