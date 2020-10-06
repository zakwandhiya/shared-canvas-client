<template>
  <v-app>
    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue';
import VueRouter from 'vue-router'

export default Vue.extend({
  name: 'App',
});
</script>
