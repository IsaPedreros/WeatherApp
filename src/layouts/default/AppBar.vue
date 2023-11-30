<template>
  <v-app-bar app dark class="bg-secondary px-8">

    <RouterLink 
    :to="{ name:'Home' }"
    style="text-decoration: none; color: inherit;"
    >
      <v-app-bar-title class="text-white">
        <v-icon @click="resetValues" icon="mdi-weather-partly-cloudy" />
        <span class="font-weight-light">Weather</span>
        <span>App</span>
      </v-app-bar-title>
    </RouterLink>


    <v-container class="mx-8">
      <v-autocomplete
        v-model="searchfield"
        v-model:search="searchQuery"
        @update:search="getSearchResults"
        return-object
        :items="mapboxSearchResults"
        item-title="place_name"
        label="Buscar una ciudad"
        variant="underlined"
        clearable
        prepend-inner-icon="mdi-magnify"
        menu-icon=""
        hide-no-data
        theme="dark"
        single-line
      ></v-autocomplete>
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

    <v-btn icon="mdi-plus" @click="addCity" v-if="route.query.preview"></v-btn>
  </v-app-bar>
</template>

<script setup>
  import { RouterLink, useRouter, useRoute } from 'vue-router';
  import { ref } from 'vue';
  import axios from "axios";
  import { uid } from "uid";

  const dialog = ref(false);

  const mapboxAPIKey = "pk.eyJ1IjoiaXNhcGVkcmVyb3MiLCJhIjoiY2xvemgya2E0MDBudDJrczFqczB6dnJtaiJ9.ETqKr1Gv-vFyMIvFVSKn-w";
  const searchQuery = ref('');
  const queryTimeout = ref(null);
  const mapboxSearchResults = ref([]);
  const selectedCity = ref([]);
  const searchError = ref(null);
  const savedCities = ref([]);
  const searchfield = ref('');


  const getSearchResults = async () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async () => {
      if (searchQuery.value !== '') {
        try {
          const result = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
          );
          mapboxSearchResults.value = result.data.features;
          selectedCity.value = mapboxSearchResults.value
            .filter(e => e.place_name === searchQuery.value)
            .map(e => {
              const [city, state] = e.place_name.split(", ");
              return {
                city,
                state,
                coordX: e.center[0],
                coordY: e.center[1],
              };
            });
          // se llama previewCity solo si hay valores asignados a selectedCity
          if (selectedCity.value && selectedCity.value.length > 0) {
            await previewCity(selectedCity.value);
          }
        } catch (error) {
          console.error('Error fetching search results:', error);
          searchError.value = true;
        }
        return;
      } else {
          resetValues();
          console.log("searchQuery es '' --");
        };
    }, 500);
  };


  const resetValues = () => {
    console.log("resetValues()--");
    mapboxSearchResults.value = [];
    selectedCity.value = [];
    searchfield.value = '';
    searchQuery.value = '';
  };

  const router = useRouter();
  const route = useRoute();

  const previewCity = async (selectedCity) => {
    console.log("previewCity -- ");
    console.log(selectedCity);

    // 1° se hace una asignacion a otra variable ya que selected city viene con reactividad de vue3 Proxy(Array), que dificultan la asignacion directa de sus propiedades.
    const datosCity = selectedCity[0]

    try {
      await router.push({
        name: "cityView",
        params: { state: datosCity.state, city: datosCity.city},
        query: {
          lat: datosCity.coordY,
          lon: datosCity.coordX,
          preview: true,
        },
      });
      console.log("router push --");
    } catch (err) { console.log("Error con el router",err)};
  };



  const addCity = () => {
    if (localStorage.getItem('savedCities')){
      savedCities.value = JSON.parse(localStorage.getItem('savedCities'));
    }

    const locationObject = {
      id: uid(),
      state: route.params.state,
      city: route.params.city,
      coords: {
        lat: route.query.lat,
        lon: route.query.lon,
      },
    };

    savedCities.value.push(locationObject);
    localStorage.setItem('savedCities', JSON.stringify(savedCities.value));

    let query = Object.assign({}, route.query);
    delete query.preview;
    router.replace({ query });
  };

</script>
