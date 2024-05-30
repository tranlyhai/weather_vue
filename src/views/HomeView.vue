<template>
  <main class="container text-white">
    <div class="relative pt-4 mb-8">
      <input 
        type="text"
        v-model="searchQuery"
        @input="getSearchResults()"
        placeholder="Search for a city or state"
        class="w-full px-1 py-2 bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
        <ul 
          class="absolute w-full px-1 py-2 text-white shadow-md bg-weather-secondary top-[66px]"
          v-if="mapboxSearchResults">
          <p v-if="searchError">
            Sorry, Something went wrong, please try again
          </p>
          <p v-if="!serverError && mapboxSearchResults.length === 0">
            No results match your query, try a different term.
          </p>
          <template v-else>
            <li
              v-for="searchResult in mapboxSearchResults" :key="searchResult.id"
              class="py-2 cursor-pointer"
              @click="previewCity(searchResult)"
              >
              {{ searchResult.properties.full_address }}
            </li>
          </template>
        </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <city-list />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>


<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import CityList from '@/components/CityList.vue';
import CityCardSkeleton from '@/components/CityCardSkeleton.vue';

const mapboxAPIKey = import.meta.env.VITE_MAPBOX_API_KEY;
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);
const serverError = ref(null);
const router = useRouter();

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout( async () => {
    if(searchQuery.value !== '') {
      try {
        const results = await axios.get(
          `https://api.mapbox.com/search/geocode/v6/forward?q=${searchQuery.value}&access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = results.data.features;
        serverError.value = false;
      } catch (error) {
        searchError.value = true;
        serverError.value = true;
      }
      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};

const previewCity = (searchResult) => {
  const [city, state] = searchResult.properties.full_address.split(",");
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    }
  });
};
</script>

