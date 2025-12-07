<script setup lang="ts">
import Stool from './components/Stool.vue';
</script>

<template>

  <div class="main_container">
    <div class="main_description">
      HERR
    </div>
    <Stool />
    <div class="main_description">
      STUHL
    </div>
    <div>
      <a href="https://dakantz.at" target="_blank">dakantz.at</a>
    </div>
  </div>
</template>

<style>
@font-face {
  font-family: "IBM Plex Mono";
  src: url('/fonts/IBM_Plex_Mono/IBMPlexMono-Medium.ttf') format('truetype');
}
.main_container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

body {
  font-family: "IBM Plex Mono", serif;
  font-optical-sizing: auto;
  font-weight: 200;
  font-style: normal;
  padding: 0;
  background-color: #9da9b7;
  margin: 0;
  display: flex;

}

.main_description {
  color: #f3fff5;
}

a {
  color: #f3e6ff;
  text-decoration: underline dotted;
}

a:hover {
  text-decoration: underline dashed;
}
</style>
