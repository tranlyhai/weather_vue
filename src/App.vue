<template>
  <div class="flex flex-col min-h-screen font-Roboto bg-weather-primary">
    <site-navigation />
    <router-view v-slot="{ Component }">
      <Transition name="page" mode="out-in">
        <component :is="Component" />
      </Transition>
    </router-view>
  </div>
</template>

<script setup>
  import { RouterView } from "vue-router";
  import SiteNavigation from "./components/SiteNavigation.vue";
</script>

<style>
.page-enter-active,
.page-leave-active {
  transition: all 600ms ease;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
}
</style>
