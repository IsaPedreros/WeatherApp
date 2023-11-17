<template>
  <v-app-bar app dark class="bg-secondary px-8">

    <RouterLink 
    :to="{ name:'Home' }"
    style="text-decoration: none; color: inherit;"
    >
      <v-app-bar-title class="text-white">
        <v-icon icon="mdi-weather-partly-cloudy" />
        <span class="font-weight-light">Weather</span>
        <span>App</span>
      </v-app-bar-title>
    </RouterLink>

    <v-container class="mx-8">
      <v-text-field
        v-model="searchQuery"
        @input="getSearchResults"
        label="Buscar una ciudad"
        variant="underlined"
        clearable
        append-inner-icon="mdi-magnify"
        single-line
      ></v-text-field>

      <!-- quizas con v-menu este configurado para que los elementos se superpongan al body (?) -->
      <v-list
        class="mt-1"
      >
        <v-list-item
          v-for="searchResult in mapboxSearchResults"
          :key="searchResult.id"
        >
        {{ searchResult.place_name }}
        </v-list-item>
      </v-list>
    </v-container>

    <v-btn 
      icon="mdi-information"
      @click="dialog = true"
    >
    </v-btn>

    <v-dialog
        v-model="dialog"
        width="auto"
      >
        <v-card color="secondary">
          <v-card-title>
            Sobre WeatherApp:
          </v-card-title>
          <v-card-text class="mb-6">
            WeatherApp te permite conocer el estado actual y futuro del clima en las ciudades que tu indiques.
          </v-card-text>
          <v-spacer></v-spacer>
          <v-card-title>
            Como usar:
          </v-card-title>
          <v-card-text class="mb-6">
            <div>1. Busca una ciudad introduciendo su nombre en la barra de búsqueda.</div>
            <div>2. Selecciona la ciudad dentro de los resultados de búsqueda. Esto te llevara al clima actual de dicha ciudad.</div>
            <div>3. Añade la ciudad clickeando en el icono "+" en la parte superior de la página. Esto guardara dicha ciudad, para que la veas en el futuro en la página de inicio.</div>
          </v-card-text>
          <v-card-title>
            Quitar una ciudad:
          </v-card-title>
          <v-card-text class="mb-6">
            Si quieres quitar una de las ciudades que añadiste a la página de inicio, en la parte inferior de la página encontrarás una opción para eliminar dicha ciudad.
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" block @click="dialog = false">Cerrar</v-btn>
          </v-card-actions>
        </v-card>
    </v-dialog>

    <v-btn icon="mdi-plus"></v-btn>
  </v-app-bar>
</template>

<script setup>
  import { RouterLink } from 'vue-router';
  import { ref } from 'vue';
  import axios from "axios";

  const dialog = ref(false);

  const mapboxAPIKey = "pk.eyJ1IjoiaXNhcGVkcmVyb3MiLCJhIjoiY2xvemgya2E0MDBudDJrczFqczB6dnJtaiJ9.ETqKr1Gv-vFyMIvFVSKn-w";
  const searchQuery = ref("");
  const queryTimeout = ref(null);
  const mapboxSearchResults = ref(null);

  const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async () => {
      if (searchQuery.value !== '') {
        try {
          const result = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
          );
          mapboxSearchResults.value = result.data.features;
          console.log(mapboxSearchResults.value);
        } catch (error) {
          console.error('Error fetching search results:', error);
        }
      return;
      } else {
        mapboxSearchResults.value = null;
      }
    }, 300);
  };

</script>