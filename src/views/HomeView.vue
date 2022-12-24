<script setup>
    import { ref } from 'vue'
    import axios from 'axios'
    import { useRouter } from 'vue-router'
    import CityList from '@/components/CityList.vue'
    import CityCardSkeleton from '@/components/CityCardSkeleton.vue';

    const weatherAppKey = "************************"
    const searchQuery = ref("")
    const queryTimeout = ref(null)
    const weatherLocationSearchResults = ref(null)
    const searchError = ref(null)
    const router = useRouter()


    const getSearchResults = () => {
        clearTimeout(queryTimeout.value)
        queryTimeout.value = setTimeout(async () => {
            if(searchQuery.value !== "") {
                try {
                    const response = await axios.get(`http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value},US&limit=5&appid=${weatherAppKey}`)
                    weatherLocationSearchResults.value = response.data
                    // console.log(weatherLocationSearchResults.value);                    
                } catch (error) {
                    searchError.value = true;
                }
                return
            }
            weatherLocationSearchResults.value = null
        }, 500);
    }
    // getSearchResults()

    const getCityData = (location) => {
        const state = location.state.replaceAll(" ", "")
        const city = location.name.replaceAll(" ", "")
        // console.log(location.lat);
        // router.push({path: `/${state}/${city}`})
        router.push({
            name: 'cityView',
            params: {state, city},
            query: {
                lat: location.lat,
                lon: location.lon,
                preview: true
            }
        })
    }

</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
        <input
          v-model="searchQuery"
          @input="getSearchResults"
          type="text"
          placeholder="Search for a city or state"
          class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
          <ul v-if="weatherLocationSearchResults" class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
            <p v-if="searchError">Something went wrong, please try again.</p>
            <p v-if="!searchError && weatherLocationSearchResults.length === 0">No city found with that name.</p>
            <template v-else>
                <li v-for="location in weatherLocationSearchResults" :key="location.lat" @click="getCityData(location)" class="py-2 cursor-pointer">
                    {{ location.name }}, {{ location.state }}
                </li>
            </template>
          </ul>
    </div>

    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>

  </main>
</template>
