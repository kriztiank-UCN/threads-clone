<template>
  <MainLayout>
    <!-- <div class="text-white">{{ user }}</div> -->

    <div id="IndexPage" class="w-full overflow-auto">
      <div class="mx-auto max-w-[500px] overflow-hidden">
        <div id="Posts" class="px-4 max-w-[600px] mx-auto">
          <!-- LOCAL DATA TEST - if there is posts, loop through them -->
          <!-- <div class="text-white" v-if="isPosts" v-for="post in posts" :key="post">
            {{ post }}
          </div> -->

          <!-- if there is posts, loop through them -->
          <div v-if="isPosts" v-for="post in posts" :key="post">
            <!-- post has a prop which is post and we're going to emit out of this so when one is deleted we update the post -->
            <Post :post="post" @isDeleted="posts = userStore.getAllPosts()" />
          </div>

          <div v-else>
            <!-- NO SSR FOR THIS SECTION -->
            <client-only>
              <div v-if="isLoading" class="mt-20 w-full flex items-center justify-center mx-auto">
                <div class="text-white mx-auto flex flex-col items-center justify-center">
                  <Icon name="eos-icons:bubble-loading" size="50" color="#ffffff" />
                  <div class="w-full mt-1">Loading...</div>
                </div>
              </div>
              <div v-if="!isLoading" class="mt-20 w-full flex items-center justify-center mx-auto">
                <div class="text-white mx-auto flex flex-col items-center justify-center">
                  <Icon name="tabler:mood-empty" size="50" color="#ffffff" />
                  <div class="w-full">Make the first post!</div>
                </div>
              </div>
            </client-only>
          </div>

          <!-- SELFCLOSING SPACER -->
          <div class="mt-60" />
        </div>
      </div>
    </div>
  </MainLayout>
</template>

<script setup>
import MainLayout from '~/layouts/MainLayout.vue'
// import state with pinia
import { useUserStore } from '~/stores/user'
const userStore = useUserStore()
// import supabase
const user = useSupabaseUser()

let posts = ref([])
let isPosts = ref(false)
let isLoading = ref(false)

watchEffect(() => {
  if (!user.value) {
    return navigateTo('/auth')
  }
})


// MOCK DATA
// onBeforeMount(() => {
//   posts.value = [
//     {
//       name: 'John Doe',
//       // user image
//       image: 'https://placehold.co/100',
//       text: 'This is the title',
//       // uploaded image
//       picture: 'https://placehold.co/500',
//     },
//   ]
// })

// fetch all posts from pinia stores/user.js getAllPosts
onBeforeMount(async () => {
  try {
    isLoading.value = true
    await userStore.getAllPosts()
    isLoading.value = false
  } catch (error) {
    console.log(error)
  }
})
// watch for changes in posts, let posts = ref([])
onMounted(() => {
  watchEffect(() => {
    posts.value = userStore.posts
    if (userStore.posts && userStore.posts.length >= 1) {
      isPosts.value = true
    } else {
      isPosts.value = false
    }
  })
})
// a single watch to watch for changes, sometimes
// on mobile the pwa can be a bit funny loading
// so this is kind of a fallback function.
// deep is watching inside of the object to see if anything changes
// then when something changes it re-gets all of the posts
watch(
  () => posts.value,
  () => {
    posts.value = userStore.posts
    if (userStore.posts && userStore.posts.length >= 1) {
      isPosts.value = true
    } else {
      isPosts.value = false
    }
  },
  { deep: true }
)
</script>
