<template>
    <h1 class="curCity">{{curCity}}</h1>
    <div class="container" style="height: 100vh;">
        <div class="row p-5 h-100">
            <div class="col-12 col-lg-8 pe-lg-5 pb-3 pb-lg-0 d-flex flex-column justify-content-between">
                <div class="row h-25 text-danger">
                    <div class="h-50 d-flex">
                        <router-link to="/">
                            <img src="@/assets/logo.png" alt="Logo" class="h-100">
                        </router-link>
                        <div v-if="currentData" class="d-flex h-100 flex-column">
                            <span class="display-5 text-light ms-5 text-uppercase">{{ currentData.name }}</span>
                            <span class="ms-5 text-light opacity-75">{{ getDate(currentData.dt) }}</span>
                        </div>
                    </div>
                </div>
                <div class="w-100 bg-dark text-light rounded shadow blur d-flex flex-column justify-content-around" style="--mdb-bg-opacity: 0.5;">
                    <!-- <NearPlaces v-if="currentData" :data="currentData"></NearPlaces> -->
                    <NearPlaces v-if="nearPlaceData" :data="nearPlaceData"></NearPlaces>
                </div>
                <div class="w-100 bg-dark text-light rounded shadow blur d-flex flex-column justify-content-around" style="--mdb-bg-opacity: 0.5;">
                    <BasicData v-if="currentData" :currentData="currentData"></BasicData>
                    <HourlyWeather v-if="hourlyData" :data="hourlyData"></HourlyWeather>
                </div>
            </div>
            <div class="col-12 col-lg-4 bg-light text-light rounded shadow blur" style="--mdb-bg-opacity: 0.2;">
                <AdditionalData :currentData="currentData" :dailyData="dailyData" v-on:changeCity="search($event)"></AdditionalData>
            </div>
        </div>
    </div>
</template>

<script>
import BasicData from '@/components/BasicData.vue';
import AdditionalData from '@/components/AdditionalData.vue'
import HourlyWeather from '@/components/HourlyWeather.vue'
import NearPlaces from '@/components/NearPlaces.vue'
import axios from 'axios';


const API = {
    weather: process.env.VUE_APP_WEATHER_API_URL,
    place: process.env.VUE_APP_FOURSQUARE_API_URL,
};

const ENDPOINTS = {
    weather: {
        currentData: `${API.weather}/weather`,
        dailyData: `${API.weather}/forecast`,
        hourlyData: `${API.weather}/forecast`,
    },
    place: {
        search: `${API.place}/search`
    }
}

const DEFAULT_LANG = 'en'
const DEFAULT_UNITS = 'metric'
const EMPTY_VAL = [null, undefined, ''];

export default {
    components: {
        BasicData,
        AdditionalData,
        HourlyWeather,
        NearPlaces,
    },
    data() {
        return {
            currentData: null,
            dailyData: null,
            hourlyData: null,
            nearPlaceData: null,
            curCity: null,
        }
    },
    async mounted() {
        if(
            EMPTY_VAL.includes(this.$route.query.city)
            && EMPTY_VAL.includes(this.$route.query.lon)
            && EMPTY_VAL.includes(this.$route.query.lat)
        ) {
            this.$router.push('/');
        } else {
            this.initGraph()
        }
    },
    methods: {
        search(city)
        {
            this.initGraph(city) // receive emit from child: AdditionalData
        },
        async initGraph (newCity = null)
        {
            let city = newCity === null ? this.$route.query.city: newCity;
            let lat = this.$route.query.lat;
            let lon = this.$route.query.lon;

            let param = this.prepParams(city, lat, lon);
            let modules = ['currentData', 'dailyData', 'hourlyData'];
            for (const key of modules) {
                let error = await this.populateWeatherData(key, param)
                if(error) {
                    this.$router.push('/');
                    break;
                }
            }

            await this.setCurCity();
            await this.populateNearPlaces();
        },
        async populateWeatherData (key, param) {
            let url = `${ENDPOINTS.weather[key]}?${param}`
            let [data, error] = await this.callApi(url)
            this[key] = data

            return error
        },
        async populateNearPlaces() {
            if (!EMPTY_VAL.includes(this.curCity)) {
                let url = `${ENDPOINTS.place.search}?query=${this.curCity}&near=japan`
                let config = {
                    headers: {
                        Authorization: process.env.VUE_APP_FOURSQUARE_API_KEY,
                        Accept: 'application/json'
                    }
                }
                let [res, error] = await this.callApi(url, config);
                this.nearPlaceData = res.results
                error
            }
        },
        prepParams(city = '', lat= '', lon = '') {
            const params = new URLSearchParams({
                q: city,
                lat: lat,
                lon: lon,
                appid: process.env.VUE_APP_WEATHER_API_KEY,
                lang: DEFAULT_LANG,
                units: DEFAULT_UNITS,
            });
            return params.toString();
        },
        getDate(unix) {
            let data = new Date(unix * 1000);
            return data.toLocaleString();
        },
        setCurCity () {
            if (!EMPTY_VAL.includes(this.$route.query.city)) {
                this.curCity = this.$route.query.city
            }

            if (this.currentData) {
                this.curCity = this.currentData.name
            }
        },
        async callApi(url, config = null)
        {
            let data = null;
            let error = false;
            await axios.get(url, config)
                .then(response => {
                    data = response.data;
                })
                .catch(function() {
                    error = true;
            });

            return [data, error]
        }
    }
}
</script>

<style scoped>
    .blur {
        backdrop-filter: blur(10px);
    }
</style>