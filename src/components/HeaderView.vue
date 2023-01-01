<script setup>
import { RouterLink } from 'vue-router'
</script>

<template>
  <nav class="fixed left-0 top-0 w-full bg-white z-10 border-b shadow-xl">
    <div class="mx-auto max-w-7xl px-6">
      <div class="relative flex items-center justify-between">

        <div class="relative py-3 flex justify-start items-center">
          <img alt="Quran logo" class="logo" src="@/assets/quran.png" width="50" />
          <span class="ml-3 text-lg font-bold">Quran App</span>
        </div>

        <div class="flex space-x-0 sm:space-x-4">
          <RouterLink to="/" class="text-gray-700 px-2 py-2 text-lg font-bold">Home</RouterLink>
          <RouterLink to="/about" class="text-gray-700 px-2 py-2 text-lg font-bold">About</RouterLink>
        </div>

      </div>
    </div>
  </nav>
  <div class="gap h-12"></div>
</template>