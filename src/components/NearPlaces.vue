<template>
    <div v-if="data" class="w-100 p-3 text-light">
        <div class="input-group">
            <h3  class="col-md-11">Near Places</h3>
            <div class="col-md-1 input-group-text">
                <i v-if="!show" v-on:click="show=!show" class="fa fa-angle-down pull-right"></i>
                <i v-if="show" v-on:click="show=!show" class="fa fa-angle-up pull-right"></i>
            </div>
        </div>
        <div v-if="show" class="near-places-content list-group">
            <a
                v-for="d in data"
                :href="'/weather?lat='+d.geocodes.main.latitude+'&lon='+d.geocodes.main.longitude"
                :key="d.lon"
                class="option-place list-group-item"
            >
                {{d.name}}
            </a>
        </div>
    </div>
</template>

<script>

export default {
    data () {
        return {
            show:null
        }
    },
    props: {
        data: Object
    },
    method: {
        toggle() {
            return this.show;
        }
    },
    mounted() {
    }
}
</script>

<style scoped>
.list-group-item {
    position: relative;
    display: block;
    padding: 0.5rem 1.5rem;
    color: #eee;
    text-decoration: none;
    background-color: transparent;
    /* border: 1px solid rgb(0 0 0 / 86%); */
}
.input-group-text {
    border: none;
    color:white;
}
</style>