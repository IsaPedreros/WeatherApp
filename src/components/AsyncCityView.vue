<template>
    <div>

    </div>
</template>

<script setup>
    import axios from 'axios';
    import { useRoute } from 'vue-router';

    const route = useRoute();

    const getWeatherData = async () => {
        try {
            const weatherData = await axios.get(`https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.coordX}&lon=${route.query.coordY}&exclude={part}&appid=8242333c05f55cc366a5ad0e8a1f189e`);

            //calculo de hora y fecha actual
            const localOffset = new Date().getTimezoneOffset() * 60000;
            const utc = weatherData.data.current.dt * 1000 + localOffset;
            weatherData.date.currentTime = utc + 1000 * weatherData.data.timezone_offset;

            //calculo de time offset por hora
            weatherData.data.hourly.forEach((hour) => {
                const utc = hour.dt * 1000 + localOffset;
                hour.currentTime = 
                    utc + 1000 * weatherData.data.timezone_offset;
            });

            return weatherData;
        } catch(err) {
            console.log(err);
        }
    };

    const weatherData = await getWeatherData();
    console.log("weatherdata --- ");
    console.log(weatherData);

</script>