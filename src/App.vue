<script setup>
import { ref } from "vue";
import _ from "lodash";

const weekday = {
  0: "Sunday",
  1: "Monday",
  2: "Tuesday",
  3: "Wednesday",
  4: "Thursday",
  5: "Friday",
  6: "Saturday",
};

const city = ref("berlin");
const cityArray = ref([]);
const dayArray = ref([]);
const weatherArray = ref([]);

const getLocation = async () => {
  const cities = await fetch(
    `https://geocoding-api.open-meteo.com/v1/search?name=${city.value}&count=10&language=en&format=json`
  );
  const data = await cities.json();
  cityArray.value = data.results;
  weatherArray.value = [];
  dayArray.value = [];
};

const drawDay = async (id) => {
  const currentCity = cityArray.value.find((city) => city.id === id);
  const temp = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${currentCity.latitude}&longitude=${currentCity.latitude}&hourly=temperature_2m`
  );

  const data = await temp.json();
  const zipData = _.zip(data.hourly.time, data.hourly.temperature_2m);
  const chankData = _.chunk(zipData, 24);
  dayArray.value = chankData;
};

const drawWeather = (day) => {
  weatherArray.value = day;
};
</script>

<template>
  <div class="app">
    <input class="input" placeholder="Enter your city" v-model="city" />
    <button class="btn" @click="getLocation">Click</button>
    <div class="cityCardWrapper">
      <div
        @click="() => drawDay(city.id)"
        class="cityCard"
        v-for="city of cityArray"
        :key="city.id"
      >
        <p>{{ city.name }}</p>
        <p>{{ city.country }}</p>
      </div>
    </div>
    <div class="daysWrapper">
      <div
        @click="() => drawWeather(day)"
        class="daysCard"
        v-for="day of dayArray"
        :key="day[0][0]"
      >
        <h2>{{ new Date(day[0][0]).getDate() }}</h2>
        <p>{{ weekday[new Date(day[0][0]).getDay()] }}</p>
      </div>
    </div>
    <table class="weatherWrapper">
      <tbody>
        <tr v-for="weather of weatherArray" :key="weather[0]">
          <td>{{ `${new Date(weather[0]).getHours()}:00` }}</td>
          <td>{{ `${weather[1]} °C` }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.app {
  width: 1200px;
  min-height: 100vh;
  margin: 0 auto;
  background: #0f2027;
  background: linear-gradient(to right, #2c5364, #203a43, #0f2027);
  padding: 20px 50px;
}

.input {
  color: #333;
  font-size: 1.2rem;
  margin: 0 auto;
  padding: 1.5rem 2rem;
  border-radius: 0.2rem;
  background-color: rgb(255, 255, 255);
  border: none;
  width: 40%;
  display: block;
  border-bottom: 0.3rem solid transparent;
  transition: all 0.3s;
}

.btn {
  text-decoration: none;
  display: block;
  width: 140px;
  height: 45px;
  border-radius: 45px;
  margin: 10px auto;
  font-size: 11px;
  text-transform: uppercase;
  text-align: center;
  letter-spacing: 3px;
  font-weight: 600;
  color: #524f4e;
  background: white;
  transition: 0.3s;
  cursor: pointer;
}

.btn:hover {
  background: #2ee59d;
  color: white;
  transform: translateY(-7px);
}

.cityCardWrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  border-bottom: 2px solid white;
  margin: 20px 0;
}

.cityCard {
  min-width: 30%;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: white;
  margin-bottom: 10px;
  cursor: pointer;
}

.daysWrapper {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.daysCard {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: white;
  width: 14%;
  user-select: none;
  cursor: pointer;
}

table {
  color: white;
  font-size: 30px;
  width: 60%;
  margin: 20px auto;
}

tr {
  display: flex;
  justify-content: space-between;
  border-bottom: 2px solid white;
  padding: 10px 0;
}
</style>
