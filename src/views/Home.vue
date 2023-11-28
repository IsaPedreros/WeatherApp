<template>
  <v-main class="align-center text-center fill-height">
    <div class="d-flex flex-column">
      <Suspense>
        <CityList/>
        <template #fallback>
          Loading..
        </template>
      </Suspense>
    </div>
  </v-main>
</template>

<script setup>
import CityList from '@/components/CityList.vue';

</script>
