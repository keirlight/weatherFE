<template>
    <div class="p-0">
        <div id="chart-div">
            <canvas id="myChart" class="text-light"></canvas>
        </div>
        <div v-if="map" class="d-flex justify-content-between">
            <p v-for="val in fiveDays" :key="val">{{ Math.round(val) }}°C</p>
        </div>
    </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';
Chart.register(...registerables);

export default {
    expose:['initChart'],
    props: {
        data: Object,
    },
    data() {
        return {
            map: new Map()
        }
    },
    computed: {
        fiveDays() {
            let temp = [];
            const values = this.map.values();
            for(let i = 0; i < this.map.size; i++) {
                temp.push(values.next().value)
            }
            return temp;
        }
    },
    methods: {
        calculateNext5Days() {
            this.map = new Map();
            for(let val of this.data.list) {
                if(this.map.size == 5) {
                    break;
                }

                let data = new Date(val.dt * 1000).toLocaleDateString(undefined, {month: "short", day: "numeric"});
                if(!this.map.has(data)) {
                    this.map.set(data, val.main.temp)
                } else {
                    if(this.map.get(data) < val.main.temp) {
                        this.map.set(data, val.main.temp)
                    }
                }
            }
        },
        destroyChart() {
            document.querySelector("#chart-div").innerHTML = '<canvas id="myChart"></canvas>';
        },
        initChart() {
            this.destroyChart();
            this.calculateNext5Days();
            const keys = this.map.keys();
            const values = this.map.values();

            Chart.defaults.color = '#ffffff'
            const ctx = document.getElementById('myChart');
            new Chart(ctx, {
                type: 'line',
                render: function () {
                    if (this.$data._chart) {
                        this.$data._chart.destroy();
                    }
                },
                data: {
                    labels: [keys.next().value, keys.next().value, keys.next().value, keys.next().value, keys.next().value],
                    datasets: [{
                        data: [ Math.round(values.next().value),
                                Math.round(values.next().value),
                                Math.round(values.next().value),
                                Math.round(values.next().value),
                                Math.round(values.next().value)],
                        backgroundColor: '#1266F1',
                        borderColor: 'rgba(255, 255, 255, 1)',
                        color: 'rgba(255, 255, 255, 1)',
                        borderWidth: 3,
                        tension: 0.5,

                    }]
                },
                options: {
                    plugins: {
                        legend: false,
                        tooltip: {
                            enabled: false
                        }
                    },
                    scales: {
                        y: {
                            display: false
                        },
                        x: {
                            grid: {
                                display: false,
                            }
                        }
                    }
                }
            });
        }
    },
    mounted() {
        this.initChart();
    }
}
</script>

<style scoped>

</style>