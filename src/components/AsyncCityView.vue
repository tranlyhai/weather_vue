<template>
  <div class="flex flex-col items-center flex-1">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="w-full p-4 text-center text-white bg-weather-secondary"
    >
      <p>
        You are current previewing this city, click the "+" icon to start
        tracking this city.
      </p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center py-12 text-white">
      <h1 class="mb-2 text-4xl">{{ route.params.city }}</h1>
      <p class="mb-12 text-sm">
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-US", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}

        {{
          new Date(weatherData.currentTime).toLocaleTimeString("en-US", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="mb-8 text-8xl">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p>Feels like {{ Math.round(weatherData.current.feels_like) }}&deg;</p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
        alt=""
      />
    </div>

    <hr class="w-full border border-white border-opacity-10" />

    <!-- Hourly Weather -->
    <div class="w-full max-w-screen-md py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 pb-4 overflow-x-scroll hourly-custom-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col items-center gap-4"
          >
            <!-- Hour -->
            <p class="text-md whitespace-nowrap">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-US", {
                  hour: "numeric",
                })
              }}
            </p>
            <!-- Image -->
            <img
              class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt=""
            />
            <p class="text-xl">{{ Math.round(hourData.temp) }}&deg;</p>
          </div>
        </div>
      </div>
    </div>

    <hr class="w-full border border-white border-opacity-10" />

    <!-- Weekly Weather -->
    <div class="w-full max-w-screen-md py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-US", {
                weekday: "long",
              })
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
            alt=""
          />
          <div class="flex justify-end flex-1 gap-2">
            <p>H: {{ Math.round(day.temp.max) }}</p>
            <p>L: {{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white duration-150 cursor-pointer hover:text-red-500"
      @click="removeCity()"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();
const openweatherAPIKey = "3d2f3e8cc0b925fb49365b9f202527a0";
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}&lon=${route.query.lng}&units=imperial&appid=${openweatherAPIKey}`
    );
    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });

    await new Promise((res) => setTimeout(res, 1000));

    return weatherData.data;
  } catch (error) {
    console.log(error);
  }
};
const weatherData = await getWeatherData();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updateCity = cities.filter((city) => city.id !== route.query.id );
  localStorage.setItem("savedCities", JSON.stringify(updateCity));
  router.push({
    name: "home",
  })
};
</script>

<style scoped>
.hourly-custom-scroll::-webkit-scrollbar {
  height: 8px;
}
.hourly-custom-scroll::-webkit-scrollbar-track {
  box-shadow: inset 0px 0px 5px rgba(0, 0, 0, 0.5);
  border-radius: 5px;
}
.hourly-custom-scroll::-webkit-scrollbar-thumb {
  background: #004E71;
  border-radius: 5px;
  cursor: pointer;
}
.hourly-custom-scroll::-webkit-scrollbar-thumb:hover {
  opacity: 0.7;
}
</style>
