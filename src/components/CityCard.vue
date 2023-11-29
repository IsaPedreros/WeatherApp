<template>
    <div>
        <v-container>
            <v-card class="bg-secondary rounded d-flex" hover>
                <v-container class="text-left">
                    <v-card-title class="text-h6">
                        {{ city.city }}
                    </v-card-title>
                    <v-card-text class="text-subtitle-1">
                        {{ city.state }}
                    </v-card-text>
                </v-container>
                <v-container class="text-right">
                    <v-card-title class="text-h5">
                        {{ Math.round(city.weather.main.temp) }}&deg; C
                    </v-card-title>
                    <v-card-text class="text-subtitle-2">
                        {{ Math.round(city.weather.main.temp_max) }}&deg; / 
                        {{ Math.round(city.weather.main.temp_min) }}&deg;
                    </v-card-text>
                </v-container>
            </v-card>
        </v-container>
    </div>
</template>

<script setup>
defineProps({
    city: {
        type: Object,
        default: () => {},
    },
});
</script>