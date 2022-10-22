<template>
    <div class="container" style="height: 100vh;">
        <div class="row p-3 h-100 d-flex align-items-center">
            <div class="col-2 col-lg-4"></div>
            <div class="col-8 col-lg-4 bg-dark text-light p-4 h-75 rounded shadow d-flex justify-content-between flex-column align-items-center" style="--mdb-bg-opacity: 0.7;">
                <div class="w-100 text-center mb-3">
                    <img src="@/assets/sun.png" alt="Logo" class="mx-auto" style="height: 100px;">
                    <p class="text-warning fw-bold">WEATHER APP</p>
                    <hr>
                </div>
                <div class="w-75 text-center">
                    <div class="form-outline">
                        <div class="text-start text-muted">Enter name of the City</div>
                        <div class="input-group">

                                <input class="form-control bg-light text-warning"
                                type="text"
                                v-on:keyup.enter="showWeather"
                                v-debounce:250="initPlaceSearch"
                                :debounce-events="['keypress']"
                                v-model="search"
                                placeholder="search"
                                aria-label="Search"
                                style="--mdb-bg-opacity: 0.3;"
                            >
                            <div class="input-group-append">
                            <button type="button" class="btn btn-outline-warning input-group-text" data-mdb-ripple-color="warning" @click="showWeather"><i class="fa-solid fa-magnifying-glass"></i></button>
                            </div>
                        </div>
                        <div class="nearPlaces dropdown">
                            <div class="dropdown-content">
                                <a
                                    v-for="d in searchData"
                                    :href="'/weather?lat='+d.geocodes.main.latitude+'&lon='+d.geocodes.main.longitude"

                                    placeholder="Near places"
                                    :key="d.lon"
                                    class="option-place"
                                    :data-name="d.name"
                                >
                                    {{d.name}}
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="w-100 text-end">
                    <hr>
                    <span class="text-muted">{{now}}</span>
                </div>
            </div>
            <div class="col-2 col-lg-4"></div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import { vue3Debounce } from 'vue-debounce';

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

export default {
    directives: {
        debounce: vue3Debounce({ lock: true })
    },
    data() {
        return {
            search: null,
            now: null,
            array: [],
            show: false
        }
    },
    computed: {
        searchData() {
            if (['', null].includes(this.search) && this.array.length === 0) return !this.show;

            if (this.search === '') {
                document.querySelector('.dropdown-content').style.display = 'none';
                return [];
            } else {
                document.querySelector('.dropdown-content').style.display = '';
            }

            return this.array.filter((d) => {
                return d.name
            });
        },
    },
    methods: {
        showWeather(lat='', lon='') {
            if(this.search != null) {
                this.$router.push('/weather?city=' + this.search)
            }
            if (lat != '' && lon != '')
            {
                return this.$router.push(`/weather?lat=${lat}&lon=${lon}`)
            }
        },
        today() {
            var today = new Date();
            var dd = String(today.getDate()).padStart(2, '0');
            var mm = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!
            var yyyy = today.getFullYear();

            this.now = mm + '/' + dd + '/' + yyyy;
        },
        async initPlaceSearch() {
            let city = this.search

            if (city != '' && city != null) {
                let url = `${ENDPOINTS.place.search}?query=${city}&near=japan`
                let config = {
                    headers: {
                        Authorization: process.env.VUE_APP_FOURSQUARE_API_KEY,
                        Accept: 'application/json'
                    }
                }
                let [res, error] = await this.callApi(url, config);
                this.array = res.results
                error
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
    },
    mounted () {
        document.querySelector('.dropdown-content').style.display = 'none';
        this.today()
    }
}
</script>

<style scoped>

.dropdown {
  position: relative;
  /* display: inline-block; */
}

.dropdown-content {
  height: auto;
  width: 100%;
  max-height: 200px;
  overflow-x: hidden;
  position: absolute;
  background-color: #f6f6f654;
  min-width: 230px;
  overflow: auto;
  border: 1px solid #f6f6f654;
  z-index: 1;
  border-radius: 0.25rem
}

.dropdown-content a {
  color: black;
  padding: 12px 16px;
  text-decoration: none;
  display: block;
}

</style>