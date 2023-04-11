<template>
  <v-app>
    <Navbar @searching="(e) => searchString = e"/>
    <v-main class="main">
      <!-- Main content -->
      <MediaGrid :searchString="searchString"/>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Navbar from './components/Navbar.vue';
import MediaGrid from './components/MediaGrid.vue';
import { ref, watch } from 'vue';

export default {
  name: 'App',  
  
  components: {
    Navbar,
    MediaGrid
  },

  setup() {
    let searchString = ref('');

    watch(
      searchString,
      (newValue) => {
        console.log(newValue)
      }
      );

    return {
      searchString,
    }
  },
};
</script>

<style scoped>
.main{
  background: #242424;
}

</style>
