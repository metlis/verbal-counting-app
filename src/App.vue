<template>
  <v-app>
    <v-content>
      <TrainerContent/>
    </v-content>
  </v-app>
</template>

<script>
import TrainerContent from './components/TrainerContent.vue';

export default {
  name: 'App',

  components: {
    TrainerContent,
  },
};
</script>
