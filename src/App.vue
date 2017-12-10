<style>
  #app {
    font-family: 'Roboto', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }

  main {
    max-width: 1200px;
    margin: 0 auto;
  }
</style>

<template>
  <v-app id="app">
    <v-content>
      <main>
        <router-view/>
      </main>
    </v-content>
  </v-app>
</template>

<script>
  export default {
    name: 'app',
  };
</script>
