<template>
  <div id="app">
    <div style="text-align:-webkit-center"><Colorcircle @change="handelChange"/></div>
    <div class="tips" v-text="tips"></div>
  </div>
</template>

<script>
import Colorcircle from './components/colorcircle'

export default {
  name: 'app',
  data() {
    return {
      tips: '',
      myId: ''
    }
  },
  components: {
    Colorcircle
  },
  methods: {
    handelChange(str) {
      // console.log(str)
      this.tips = str
      this.$el.querySelector('.tips').style.backgroundColor = str
    }
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
.tips{
  margin:10px;
}
</style>
