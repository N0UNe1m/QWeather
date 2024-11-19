<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input v-model="search" @keyup.enter="getWeatherBySearch" placeholder="Search" dark borderless>
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>
        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.location.name }}</div>
        <div class="text-h6 text-weight-light">{{ weatherData.current.condition.text }}</div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.current.temp_c) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="weatherData.current.condition.icon" />
      </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Weather<br />Widget</div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col skyline" />
  </q-page>
</template>

<script>
import { ref, computed } from "vue";
import { api } from "boot/axios";
import { useQuasar } from "quasar";

export default {
  name: "IndexPage",
  setup() {
    const $q = useQuasar();
    const search = ref("");
    const weatherData = ref(null);
    const lat = ref(null);
    const lon = ref(null);

    const getLocation = () => {
      navigator.geolocation.getCurrentPosition((position) => {
        lat.value = position.coords.latitude;
        lon.value = position.coords.longitude;
        getWeatherByCoords();
      }, (error) => {
        $q.notify({
          color: 'negative',
          message: `Error getting location: ${error.message}`,
          icon: 'report_problem'
        });
      });
    };

    const getWeatherByCoords = () => {
      api.get(`/current.json?key=c3c3e42644a546e5803175413241811&q=${lat.value},${lon.value}`)
        .then((response) => {
          weatherData.value = response.data;
        })
        .catch((error) => {
          $q.notify({
            color: 'negative',
            message: `Error: ${error.message}`,
            icon: 'report_problem'
          });
        });
    };

    const getWeatherBySearch = () => {
      api.get(`/current.json?key=c3c3e42644a546e5803175413241811&q=${search.value}`)
        .then((response) => {
          weatherData.value = response.data;
        })
        .catch((error) => {
          $q.notify({
            color: 'negative',
            message: `Error: ${error.message}`,
            icon: 'report_problem'
          });
        });
    };

    const bgClass = computed(() => {
      if (weatherData.value && weatherData.value.current.is_day) {
        return "bg-day";
      } else {
        return "bg-night";
      }
    });

    return { search, weatherData, getLocation, getWeatherBySearch, bgClass };
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.my_image
  width: 30%
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../skyline.png)
  background-size: contain
  background-position: center bottom
</style>
