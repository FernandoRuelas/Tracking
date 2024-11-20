<template>
  <v-app>
  <div id="app">
    <div class="pa-2 d-flex">
      <h3>CRUD with maps</h3>
      <v-spacer></v-spacer>
      <v-btn icon color="primary" id="recordar" @click="clearTourAndRefresh"><i class="fas fa-question-circle"></i></v-btn>
    </div>
    <div class="background"></div>
    <!-- <v-app>
      <router-view />
    </v-app> -->
    <div class="content">
   
        <router-view />
      </div>
    </div>
  </v-app>
  </template>

<style lang="scss">
.background {
  position: absolute;
  height: 250px;
  width: 100%;
  background: #00205c;
}

.content {
  padding: 20px 20px 0px 20px;
  position: absolute;
  width: 100%;
}


nav {
  padding: 15px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>

<script>
export default {
  methods: {
    clearTourAndRefresh() {
      localStorage.removeItem('tour');
    
    // Actualiza la página
    window.location.reload();
    }
  }
}
</script>