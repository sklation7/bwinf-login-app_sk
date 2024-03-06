<script setup>
import Login from './components/Login.vue';
import Registration from './components/Registration.vue';
import { ref } from 'vue'

let isLoggedIn=ref(false)
</script>

<template>
  <div v-if="!isLoggedIn" class="flex flex-col">
    <h1 class="text-3xl font-bold mx-auto p-4">
      This is a login dialogue
    </h1>
    <div class="flex flex-row mx-auto">
      <button @click="isLoggedIn = !isLoggedIn" class="p-4 m-4 rounded-lg font-sans font-bold text-white bg-sky-500 hover:bg-sky-700">Login</button>
      <button class="p-4 m-4 rounded-lg font-sans font-bold bg-sky-500 hover:bg-sky-700 text-white">Registration</button>
    </div>
  </div>
  <div v-if="isLoggedIn" class="flex flex-row">
    <Login class="mx-auto"></Login>
  </div>
</template>

<style scoped>
</style>
