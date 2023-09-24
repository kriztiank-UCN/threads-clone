<template>
  <div class="bg-black fixed w-[calc(100%+1px)] h-full">
    <NuxtPage />
    <!-- import CreatePost from '~/components/CreatePost.vue' -->
    <!-- 
    we are going to add a dynamic class: so what happens: if isMenuOverlay is true we are going to
    open our overlay so it covers the entire screen and if it's false we close it,
    so max-h-[100vh] when its true/open and max-h-0 when it's false/closed. 
    -->
    <CreatePost
      :class="[
        { 'max-h-[100vh] transition-all duration-200 ease-in visible': userStore.isMenuOverlay },
        { 'max-h-0 transition-all duration-200 ease-out invisible': !userStore.isMenuOverlay },
      ]"
    />
  </div>
</template>

<script setup>
import { useUserStore } from '~/stores/user'
const userStore = useUserStore()
</script>
