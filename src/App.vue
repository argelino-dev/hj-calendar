<template>
  <div id="app">
    <HjCalendar v-model="deliveryDate" @dateChanged="deliveryDateChanged"/>
  </div>
</template>

<script>
import HjCalendar from './components/HjCalendar.vue'

export default {
  name: 'App',
  components: {
    HjCalendar
  },
  data: function () {
    return {
      deliveryDate: null,
    }
  },
  methods:{
    deliveryDateChanged($newDate){
      // moment-hijri object
      console.log($newDate);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
