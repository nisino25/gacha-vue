<template>
  <span v-for="(item, index) in gachadata" :key="index">
    {{ item.name }}
  </span>
</template>

<script>
import data from "./const/data.json"

export default {
  name: 'App',
  data(){
    return{
      gachadata: data,
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
