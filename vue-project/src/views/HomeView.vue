<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';
import CityView from './CityView.vue';

const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.formatted.split(","); // splits the formatted part of the code into city, and state
  router.push({
    name: CityView,
    params: {state: state.replaceAll("", ""), city: city},
    query: {
      lat: searchResult.geometry.value,
      lng: searchResult.geometry.value,
      preview: true,
    }
  })
}

const opencageAPIKey = "bdf3abffbdfb4ac58b583b837154d4f3";
const searchQuery = ref('');
const queryTimeout = ref(null);
const opencageSearchResults = ref(null);
const apiURL = 'https://api.opencagedata.com/geocode/v1/json';
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async() => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
        `${apiURL}?key=${opencageAPIKey}&q=${searchQuery.value}&pretty=1&no_annotations=1&limit=5` // Passing the optional parameter to get one search result instead of 10
      );
      // console.log(result.data); // Checking if we are getting anything back
      opencageSearchResults.value = result.data.results;
      // console.log(opencageSearchResults.value);
      } catch {
        searchError.value = True;
      }
      return; // Otherwise, the line below will ALSO run
    }
    opencageSearchResults.value = null;
  }, 300);
};
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text"
      v-model="searchQuery"
      @input="getSearchResults"
      placeholder="Search for a city of state" class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
      <ul 
      v-if="opencageSearchResults"
      class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
      <p v-if="searchError">
        Sorry, something went wrong, please try again.
      </p>
      <p v-if="!searchError && opencageSearchResults.length === 0">
        No resutls match your query, please try a different term.
      </p>
        <template v-else>
          <li v-for="searchResult in opencageSearchResults"
          :key="searchResult.id"
          class="py-2 cursor-pointer"
          @click="previewCity(searchResult)">
          {{ searchResult.formatted }}</li>
        </template>
      </ul>
    </div>
  </main>
</template>
