<script setup lang="ts">
const config = useRuntimeConfig();
const search = ref("Berlin");
const input = ref("");

const { data: city, error } = useFetch(
  () =>
    `${config.public.apiBase}${search.value}&appid=${config.public.apiSecretKey}`
);

function handleInputClick() {
  const formatedSearch = input.value.trim().split(" ").join("+");
  search.value = formatedSearch;
  input.value = "";
}

const onEnter = () => {
  handleInputClick();
};
</script>

<template>
  <div class="h-screen overflow-hidden relative">
    <div class="absolute size-full top-0 overlay" />
    <div class="absolute size-full top-0 p-48">
      <div class="flex justify-between items-center">
        <div class="flex flex-col gap-2">
          {{ input }}
          <h1 class="text-7xl text-gray-300">
            {{ city.name }}
          </h1>
          <p class="font-extralight text-2xl text-gray-300">
            Sonntag, 20. April
          </p>
          <NuxtImg
            :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`"
            alt=""
            class="w-56 icon"
          />
        </div>
        <div>
          <p class="text-9xl text-gray-300 font-extralight">
            {{ (city.main.temp - 273, 15) }}°
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
