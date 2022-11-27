<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';
import CityList from "../components/CityList.vue";

const router = useRouter();
const searchQuery = ref("");
const QueryTimeout = ref(null);
const searchError = ref(null);
const  previewCity = (serachResult) => {
  // console.log(serachResult);
  const [city, state] = serachResult.place_name.split(",");
  console.log(city, state);
  router.push({
    name: "cityView",
    params:{ state: state.replaceAll(" ",""),city: city},
    query:{
      lat: serachResult.geometry.coordinates[1],
      lng: serachResult.geometry.coordinates[0],
      preview: true,
    },
  });
}
const mapboxSearchResults = ref(null);
const mapboxAPIKey =
    "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const getSearchResults = () => {
  clearTimeout(QueryTimeout.value);
  QueryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== ""){
      try {
        const result = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
        console.log(mapboxSearchResults.value)
      } catch{
          searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = null;
  },300)
}
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
          @input="getSearchResults"
          v-model="searchQuery"
          placeholder="Search For a city or state"
          class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
          type="text">
      <ul class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]" v-if="mapboxSearchResults">
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!searchError && mapboxSearchResults.length ===0">No result match your query, try a different term.</p>
          <template v-else>
            <li v-for="serachResult in mapboxSearchResults" :key="serachResult.id" class="py-2 cursor-pointer" @click="previewCity(serachResult)">
              {{ serachResult.place_name }}
            </li>
          </template>
      </ul>
    </div>


    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList/>
        <template #fallback>
          <CityCardSkeleton/>
        </template>
      </Suspense>
    </div>
  </main>
</template>
