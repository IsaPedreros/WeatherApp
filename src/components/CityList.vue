<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)"/>
    </div>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';

const savedCities = ref([]);
const getCities = async () => {
    if (localStorage.getItem('savedCities')) {
        savedCities.value = JSON.parse(
            localStorage.getItem('savedCities')
        );

        const requests = [];

        savedCities.value.forEach((city) => {
            requests.push(
                axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lon}&units=metric&appid=8242333c05f55cc366a5ad0e8a1f189e`)
            );
        });
        
        const weatherData = await Promise.all(requests);

        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data;
        });
    }
};

await getCities();

const router = useRouter();
const goToCityView = (city) => {
    console.log(city);
    router.push({
        name: 'cityView',
        params: { state: city.state, city: city.city },
        query: {lat: city.coords.lat, lon: city.coords.lon},
    });

};

</script>
