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
              <!-- v-if="!hasLikedComputed" show empty heart
                   v-else show full red heart -->
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
              <div>
                <!-- show how many likes for this post -->
                <span v-if="!isLike">{{ post.likes.length }}</span>
                <!-- if isLike is true show loading icon -->
                <span v-else>
                  <Icon name="eos-icons:bubble-loading" color="#ffffff" size="13" />
                </span>
                likes
              </div>
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

// LIKES #1
const hasLikedComputed = computed(() => {
  // if there's no user just return just exit
  if (!user.value) return
  let res = false
  // Loop through our props so the post object
  // we Loop through the likes inside the post
  /* because a post has multiple likes, it's a one-to-many relationship
     so get each individual like and if the like.userId == is equal to the logged in user
     and postId equals the postId being passed as a prop we want to set this to
    true and then we just return the result which is either true or false */
  props.post.likes.forEach(like => {
    if (like.userId == user.value.identities[0].user_id && like.postId == props.post.id) {
      res = true
    }
  })

  return res
})

const deletePost = async (id, picture) => {
  let res = confirm('Are you sure you want to delete this post?')
  // if it's false we just want to exit this function
  if (!res) return
  // if not we want to try to delete the post
  try {
    isMenu.value = false
    isDeleting.value = true
    const { data, error } = await client.storage.from('threads-clone-files-').remove([picture])

    await useFetch(`/api/delete-post/${id}`, { method: 'DELETE' })
    emit('isDeleted', true)

    isDeleting.value = false
  } catch (error) {
    console.log(error)
    isDeleting.value = false
  }
}

const likePost = async id => {
  isLike.value = true
  try {
    await useFetch(`/api/like-post/`, {
      method: 'POST',
      body: {
        userId: user.value.identities[0].user_id,
        postId: id,
      },
    })
    await userStore.getAllPosts()
    isLike.value = false
  } catch (error) {
    console.log(error)
    isLike.value = false
  }
}

const unlikePost = async id => {
  isLike.value = true
  try {
    await useFetch(`/api/unlike-post/${id}`, { method: 'DELETE' })
    await userStore.getAllPosts()
    isLike.value = false
  } catch (error) {
    console.log(error)
    isLike.value = false
  }
}
// TODO
/* this is where
we figure out if we're liking or unliking a post first we're going to say let like post object equals null and the
first thing we're going to say is if props post likes length is less than one
we obviously want it to be alike because there is no likes on it at all so we're going to call this like post and then
after that we're going to say else and then we've got a bit of a loop in there so we're going to grab our props post
likes for each and we can open that up we're going to say like open it and then
inside this we're going to say if like user ID equals your user ID and the post
ID equals the post ID being passed in we're going to assign that like to this variable that we've set up here and and
then finally after this else we're going to say if the like post object has something in it we want to unlike and
then we have this return null to exit the function but if this is not true we're gonna like the post and that is it */
const likesFunc = () => {
  let likePostObj = null

  if (props.post.likes.length < 1) {
    likePost(props.post.id)
    return null
  } else {
    props.post.likes.forEach(like => {
      if (like.userId == user.value.identities[0].user_id && like.postId == props.post.id) {
        likePostObj = like
      }
    })
  }

  if (likePostObj) {
    unlikePost(likePostObj.id)
    return null
  }

  likePost(props.post.id)
}
</script>
