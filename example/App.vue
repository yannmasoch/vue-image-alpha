<template>
  <div id="app">
    <img-alpha :src="require('../example/img/mario.jpg')" :src-alpha="require('../example/img/mario-alpha.jpg')"/>
    <br><br>
    <img :src="require('../example/img/mario.jpg')" />
    <img :src="require('../example/img/mario-alpha.jpg')" />

  </div>
</template>

<script>
import imgAlpha from '../src/ImgAlpha.vue'

export default {
  name: 'app',
  components: {
    imgAlpha
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

img {
  /* padding: 10px; */
  margin: 10px;
  max-width: 600px;
  /* border: 1px solid #888; */
  /* border-radius: 4px; */
}
</style>
