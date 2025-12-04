<script setup lang="ts">
import Stool from './components/Stool.vue';
</script>

<template>

  <div>
    <div class="main_description">
      HERR
    </div>
    <Stool />
  </div>
  <div class="main_description">
    STUHL
  </div>
  <div>
    <a href="https://dakantz.at" target="_blank">dakantz.at</a>
  </div>
</template>

<style>
@font-face {
  font-family: "IBM Plex Mono";
  src: url('/fonts/IBM_Plex_Mono/IBMPlexMono-Medium.ttf') format('truetype');
}

body {
  font-family: "IBM Plex Mono", serif;
  font-optical-sizing: auto;
  font-weight: 200;
  font-style: normal;
  padding: 0;
  background-color: #9da9b7;
}

.main_description {
  color: #7bd185;
}
</style>
