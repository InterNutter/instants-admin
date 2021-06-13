<template>
  <div id="app">
    <admin/>
  </div>
</template>

<script>

import admin from './components/admin.vue';
import Vue from 'vue';
import VCalendar from 'v-calendar';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import locale from 'element-ui/lib/locale/lang/en'

Vue.use(ElementUI, { locale })
Vue.use(VCalendar, {});

export default {
  name: 'App',
  components: {
    admin
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
