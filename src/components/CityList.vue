<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)"/>
  </div>

  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in the bield above.
  </p>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';

import CityCard from './CityCard.vue';

const savedCities = ref([]);

const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'));

    const request = [];
    savedCities.value.forEach((city) => {
      request.push(
        axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.cords.lat}&lon=${city.cords.lng}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=metric`)
      )
    });

    const weatherData = await Promise.all(request);

    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value.data;
    })
  }
}

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: {
      state: city.state, 
      city: city.city,
    },
    query: {
      lat: city.cords.lat,
      lng: city.cords.lng,
      id: city.id,
    }
  });
}

await getCities();
</script>
