<template>
  <v-app>
    <v-content>
      <CountingTrainer/>
    </v-content>
  </v-app>
</template>

<script>
import CountingTrainer from './components/CountingTrainer.vue';

export default {
  name: 'App',

  components: {
    CountingTrainer,
  },
};
</script>
