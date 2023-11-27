<template>
    <div class="items-center">
        <div class="text-center">
            <v-alert type="info" variant="tonal" class="text-subtitle-2" closable>Estás viendo el tiempo para {{ route.params.city }}. Si deseas agregar esta ciudad al inicio presiona el signo más (+) de arriba. </v-alert>
        </div>
        <div class="text-center py-9">
            <p class="text-h5 mb-2"> {{ route.params.city }}</p>
            <p class="text-body-2 mb-12">
                {{ 
                    new Date(weatherData.currentTime).toLocaleDateString(
                        'es-ES',
                        {
                            weekday: "short",
                            day: "numeric",
                            month: "short",
                            year: "numeric",
                        }
                    )
                }}
                {{ 
                    new Date(weatherData.currentTime).toLocaleTimeString(
                        'es-ES',
                        {
                            timeStyle: "short",
                        }
                    )
                }}
            </p>
            <p class="text-h2 mb-9">
                {{ Math.round(weatherData.current.temp) }}&deg; C
            </p>
            <p class="mb-2">
                Se sienten como 
                {{ Math.round(weatherData.current.feels_like) }}&deg; C
            </p>
            <p class="text-capitalize">
                {{ weatherData.current.weather[0].description }}
            </p>
            <v-img
                class="mx-auto mb-12"
                width="150"
                :src="`https://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@4x.png`"
            >
            </v-img>

            <v-divider></v-divider>
        </div>
        <!-- Clima diario -->
        <div class="py-6 mx-auto text-h6 text-center">
            Próximas horas:
        </div>
        <v-sheet class="mx-auto text-center bg-primary">
            <v-slide-group 
                class="pa-3"
                show-arrows
            >
                <v-slide-group-item 
                    v-for="hourData in weatherData.hourly" 
                    :key="hourData.dt" 
                >
                    <v-container
                        class="ma-4 text-subtitle-2 text-wrap"
                    >
                        <p>
                            {{ 
                                new Date(hourData.currentTime).toLocaleTimeString(
                                    'en-us', 
                                    {
                                        hour: "numeric",
                                    })
                            }}
                        </p>
                        <img 
                            class="w-auto"
                            :src="`https://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`" 
                            
                        >
                        <p>
                            {{ Math.round(hourData.temp) }}&deg; C
                        </p>
                    </v-container>
                </v-slide-group-item>
            </v-slide-group>
        </v-sheet>

        <v-divider></v-divider>
        <!-- Clima Semana -->
        <div class="py-12 mx-8 text-h6 text-center">
            Pronóstico de la semana:
        </div>
        <v-sheet class="mx-auto text-center bg-primary">
            <div
                v-for="day in weatherData.daily"
                :key="day.dt"
                class="d-flex flex-row"
            >
                <v-container class="d-inline">
                    {{ new Date(day.dt * 1000).toLocaleDateString(
                        'es-ES',
                            {
                                weekday: 'long',
                            }
                        )  
                    }}
                </v-container>
            </div>
        </v-sheet>

    </div>

</template>

<script setup>
    import axios from 'axios';
    import { useRoute } from 'vue-router';

    const route = useRoute();
    const getWeatherData = async () => {
        try {
            const weatherData = await axios.get(`https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}&lon=${route.query.lon}&units=metric&lang=es&appid=8242333c05f55cc366a5ad0e8a1f189e`);

            //calculo de hora y fecha actual
            const localOffset = new Date().getTimezoneOffset() * 60000;
            const utc = weatherData.data.current.dt * 1000 + localOffset;
            weatherData.data.currentTime = utc + 1000 * weatherData.data.timezone_offset;

            //calculo de time offset por hora
            weatherData.data.hourly.forEach((hour) => {
                const utc = hour.dt * 1000 + localOffset;
                hour.currentTime = 
                    utc + 1000 * weatherData.data.timezone_offset;
            });

            return weatherData.data;
        } catch(err) {
            console.log(err);
        }
    };

    const weatherData = await getWeatherData();
    console.log("weatherdata --- ");
    console.log(weatherData);

</script>