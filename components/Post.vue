<template>
  <div class="z-50 bottom-0 h-full w-full">
    <div class="py-2 w-full">
      <div class="flex items-center justify-between">
        <!-- BLOG POST -->
        <div class="flex items-center text-white">
          <img class="rounded-full h-[35px]" :src="post.image" />
          <div class="ml-2 font-semibold text-[18px]">{{ post.name }}</div>
        </div>

        <!-- if there is a user and user identities then we're going to say if the user ID equals the post user ID we want to show this button section because that means it's our post and we can delete it but if the user ID isn't the same as the logged in user ID then it's not our post so we can't delete it -->

        <!-- add an click event "is menu is not equal to is menu" so if it's true it becomes false, if it's false it becomes true -->
        <div
          v-if="user && user.identities && user.identities[0].user_id == post.userId"
          @click="isMenu = !isMenu"
          class="relative"
        >
          <button
            :disabled="isDeleting"
            class="flex items-center text-white p-1 h-[24px] w-[24px] hover:bg-gray-800 rounded-full cursor-pointer"
            :class="isMenu ? 'bg-gray-800' : ''"
          >
            <Icon v-if="!isDeleting" name="bi:three-dots" color="#ffffff" size="18" />
            <Icon v-else name="eos-icons:bubble-loading" color="#ffffff" size="18" />
          </button>
          <!-- if isMenu is true then show the div -->
          <div v-if="isMenu" class="absolute border border-gray-600 right-0 z-20 mt-1 rounded">
            <button
              @click="deletePost(post.id, post.picture)"
              class="flex items-center rounded gap-2 text-red-500 justify-between bg-black w-full pl-4 pr-3 py-1 hover:bg-gray-900"
            >
              <div>Delete</div>
              <Icon name="solar:trash-bin-trash-broken" size="20" />
            </button>
          </div>
        </div>
      </div>

      <div class="relative flex items-center w-full">
        <!-- LEFT BORDER -->
        <div class="w-[42px] mx-auto">
          <div class="absolute ml-4 mt-1 top-0 w-[1px] bg-gray-700 h-full" />
        </div>

        <div class="bg-black rounded-lg w-[calc(100%-50px)] text-sm w-full font-light">
          <!-- POST TITLE -->
          <div class="py-2 text-gray-300">{{ post.text }}</div>
          <!-- POST IMAGE -->
          <!-- SUPABASE BUCKET_URL IS SETUP IN nuxt.config.ts -->
          <img
            v-if="post && post.picture"
            class="mx-auto w-full mt-2 pr-2 rounded"
            :src="runtimeConfig.public.bucketUrl + post.picture"
          />

          <div class="absolute mt-2 w-full ml-2">
            <button :disabled="isLike" @click="likesFunc()" class="flex items-center gap-1">
              <Icon
                v-if="!hasLikedComputed"
                class="p-1 text-white hover:bg-gray-800 rounded-full cursor-pointer"
                name="mdi:cards-heart-outline"
                size="28"
              />
              <Icon
                v-else
                class="p-1 text-red-500 hover:bg-gray-800 rounded-full cursor-pointer"
                name="mdi:cards-heart"
                size="28"
              />
            </button>
            <div class="relative text-sm text-gray-500">
              <!-- TEMP -->
              <div>
                <span>4</span>
                likes
              </div>
              <!-- TEMP -->
              <!-- <div> 
                              <span v-if="!isLike">{{ post.likes.length }}</span>
                              <span v-else>
                                  <Icon name="eos-icons:bubble-loading" color="#ffffff" size="13"/>
                              </span>
                              likes
                          </div> -->
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- DUMMY CONTENT -->
    <div class="relative inline-block text-gray-500 pt-1 pb-1.5">
      <div class="flex items-center">
        <div class="flex items-center flex-wrap text-white gap-1 w-[42px]">
          <div class="flex gap-0.5">
            <img class="rounded-full h-[14px] mt-2" src="https://picsum.photos/id/202/50" />
            <img class="rounded-full h-[17px]" src="https://picsum.photos/id/223/50" />
          </div>
          <div class="flex items-center">
            <img class="rounded-full h-[11px] ml-4" src="https://picsum.photos/id/99/50" />
          </div>
        </div>
      </div>
    </div>
    <!-- BOTTOM BORDER -->
    <div class="h-[1px] bg-gray-800 w-full mt-3" />
  </div>
</template>

<script setup>
import { useUserStore } from '~/stores/user'
const userStore = useUserStore()
// .env
const runtimeConfig = useRuntimeConfig()

let isMenu = ref(false)
let isLike = ref(false)
let isDeleting = ref(false)
// pages/index has a prop which is post and we're going to emit out of this so when one is deleted we update the post
// <Post :post="post" @isDeleted="posts = []" />
const emit = defineEmits(['isDeleted'])
// <Post :post="post"
const props = defineProps({ post: Object })

const client = useSupabaseClient()
const user = useSupabaseUser()
</script>
