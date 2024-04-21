<script setup lang="ts">
interface WeatherData {
  coord: Coord;
  weather: Weather[];
  base: string;
  main: Main;
  visibility: number;
  wind: Wind;
  clouds: Clouds;
  dt: number;
  sys: Sys;
  timezone: number;
  id: number;
  name: string;
  cod: number;
}
interface Sys {
  type: number;
  id: number;
  country: string;
  sunrise: number;
  sunset: number;
}
interface Clouds {
  all: number;
}
interface Wind {
  speed: number;
  deg: number;
  gust: number;
}
interface Main {
  temp: number;
  feels_like: number;
  temp_min: number;
  temp_max: number;
  pressure: number;
  humidity: number;
}
interface Weather {
  id: number;
  main: string;
  description: string;
  icon: string;
}
interface Coord {
  lon: number;
  lat: number;
}

const config = useRuntimeConfig();

const cookie = useCookie("city");

if (!cookie.value) {
  cookie.value = "Berlin";
}

const search = ref(cookie.value);
const input = ref("");
const background = ref("");

// Diese Composable ist einfach
// Um komplexere Dinge zu fetchen, sollte man useAsyncData benutzen
// const { data: city, error } = useFetch(
//   () =>
//     `${config.public.apiBase}${search.value}&appid=${config.public.apiSecretKey}`
// );

//dieser Code ist zu 100% äquivalent mit useFetch() sie oben
// const { data: city, error } = useAsyncData("city", async () => {
//   const response = await $fetch(
//     `${config.public.apiBase}${search.value}&appid=${config.public.apiSecretKey}`
//   );

//   return response;
// });

const date = ref(new Date(new Date().getTime() - 25200 * 1000));

const { data: city, error } = await useAsyncData(
  "city",
  async () => {
    let response: WeatherData;
    try {
      response = await $fetch(
        `${config.public.apiBase}${search.value}&appid=${config.public.apiSecretKey}`
      );

      cookie.value = search.value;

      const temp = response.main.temp;

      if (temp - 273.15 <= -10) {
        background.value =
          "https://images.unsplash.com/photo-1483664852095-d6cc6870702d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
      } else if (temp - 273.15 > -10 && temp - 273.15 <= 0) {
        background.value =
          "https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
      } else if (temp - 273.15 > 0 && temp - 273.15 <= 10) {
        background.value =
          "https://images.unsplash.com/photo-1560258018-c7db7645254e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=4032&q=80";
      } else {
        background.value =
          "https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3546&q=80";
      }
      return response;
    } catch (error) {}
  },
  {
    watch: [search],
  }
);

function handleInputClick() {
  const formatedSearch = input.value.trim().split(" ").join("+");
  search.value = formatedSearch;
  input.value = "";
}

const onEnter = () => {
  handleInputClick();
};

const handleReturnHome = () => {
  window.location.reload();
};
</script>

<template>
  <div v-if="city" class="h-screen overflow-hidden relative">
    <img :src="background" alt="background image" />
    <div class="absolute size-full top-0 overlay" />
    <div class="absolute size-full top-0 p-48">
      <div class="flex justify-between items-center">
        <div class="flex flex-col gap-2">
          <h1 v-if="city" class="text-7xl text-gray-300">
            {{ city.name }} - {{ city.sys.country }}
          </h1>
          <p class="font-extralight text-2xl text-gray-300">
            {{ date }}
          </p>
          <img
            :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`"
            alt=""
            class="w-56 icon"
          />
          <p class="text-neutral-200 capitalize text-xl">
            {{ city.weather.map((d) => d.description).toString() }}
          </p>
        </div>
        <div>
          <p class="text-9xl text-gray-300 font-extralight">
            {{ (city.main.temp - 273.15).toFixed() }}°
          </p>
        </div>
      </div>
      <div class="mt-20">
        <input
          type="text"
          class="w-1/2 h-10 rounded py-2 px-4"
          placeholder="Stadt..."
          v-model="input"
          @keyup.enter="onEnter"
        />
        <button
          @click="handleInputClick"
          class="bg-sky-500 w-30 text-gray-300 h-10 py-2 px-4 rounded"
        >
          Suchen
        </button>
      </div>
    </div>
  </div>
  <div
    v-else
    class="flex flex-col gap-4 justify-center items-center h-screen bg-slate-900"
  >
    <h3 class="text-4xl text-gray-400">Etwas ist schiefgelaufen</h3>
    <p
      @click="handleReturnHome"
      class="rounded px-4 py-2 text-4xl text-gray-400 cursor-pointer hover:bg-yellow-500 duration-300 transition-colors hover:text-white"
    >
      zurück
    </p>
  </div>
</template>

<style scoped>
.overlay {
  background-color: rgb(0, 0, 0, 0.6);
}
.icon {
  margin-left: -2.5rem;
  margin-top: -2.75rem;
}
</style>
