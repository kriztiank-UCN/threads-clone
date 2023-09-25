<template>
  <div id="CreatePost" class="fixed z-50 bottom-0 h-full w-full overflow-hidden">
    <div class="bg-black h-full w-full text-white overflow-auto">
      <div
        class="flex items-center justify-start py-4 max-w-[500px] mx-auto border-b border-b-gray-700"
      >
        <!-- CLOSE MENU OVERLAY -->
        <button @click=";(userStore.isMenuOverlay = false), clearData()" class="rounded-full px-2">
          <Icon name="mdi:close" size="25" />
        </button>
        <div class="text-[16px] font-semibold">New Thread</div>
      </div>

      <div id="Post" class="z-40 bottom-0 max-h-[100vh-200px] w-full px-3 max-w-[500px] mx-auto">
        <div class="py-2 w-full">
          <div class="flex items-center">
            <div v-if="user" class="flex items-center text-white">
              <img
                class="rounded-full h-[35px]"
                :src="user.identities[0].identity_data.avatar_url"
              />
              <div class="ml-2 font-semibold text-[18px]">
                {{ user.identities[0].identity_data.full_name }}
              </div>
            </div>
          </div>
          <!-- DYNAMIC TEXTAREA -->
          <div class="relative flex items-center w-full">
            <div class="w-[42px] mx-auto">
              <div class="absolute ml-4 mt-1 top-0 w-[1px] bg-gray-700 h-full" />
            </div>
            <div class="bg-black rounded-lg w-[calc(100%-50px)] text w-full font-light">
              <div class="pt-2 text-gray-300 bg-black w-full">
                <textarea
                  v-model="text"
                  style="resize: none"
                  placeholder="Start a thread..."
                  id="textarea"
                  @input="adjustTextareaHeight()"
                  class="w-full bg-black outline-none"
                ></textarea>
              </div>
              <!-- FILE UPLOAD... -->
              <div class="w-full">
                <div class="flex flex-col gap-2 py-1">
                  <!-- if fileDisplay has something in it we want to display this image and we've got a source which is fileDisplay and then underneath that div we have a label that is going to be our file input -->
                  <div v-if="fileDisplay">
                    <img class="mx-auto w-full mt-2 mr-2 rounded-lg" :src="fileDisplay" />
                  </div>

                  <label for="fileInput">
                    <Icon name="clarity:paperclip-line" color="#ffffff" size="25" />
                    <input
                      ref="file"
                      type="file"
                      id="fileInput"
                      @input="onChange"
                      hidden
                      accept=".jpg,.jpeg,.png"
                    />
                  </label>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- if there is some text show button -->
      <button
        v-if="text"
        :disabled="isLoading"
        @click="createPost"
        class="fixed bottom-0 font-bold text-lg w-full p-2 bg-black inline-block float-right p-4 border-t border-t-gray-700"
        :class="isLoading ? 'text-gray-600' : 'text-blue-600'"
      >
        <!-- INSIDE BUTTON SHOW A POST OR THE LOADING STATE-->
        <div v-if="!isLoading">Post</div>
        <div v-else class="flex items-center gap-2 justify-center">
          <Icon name="eos-icons:bubble-loading" size="25" />
          Please wait...
        </div>
      </button>
    </div>
  </div>
</template>

<script setup>
import { v4 as uuidv4 } from 'uuid'
import { useUserStore } from '~/stores/user'
const userStore = useUserStore()

const client = useSupabaseClient()
const user = useSupabaseUser()

let text = ref(null)
let isLoading = ref(false)

// text area changes height based on how many characters or words you have inside of it
// @input="adjustTextareaHeight()" add as a property to the textarea
const adjustTextareaHeight = () => {
  var textarea = document.getElementById('textarea')
  textarea.style.height = 'auto'
  textarea.style.height = textarea.scrollHeight + 'px'
}

let file = ref(null)
let fileDisplay = ref(null)
let fileData = ref(null)

const clearData = () => {
  text.value = null
  file.value = null
  fileDisplay.value = null
  fileData.value = null
}
// onChange: when we upload the image we can save it and then convert it into a readable URL so we can see it
const onChange = () => {
  fileDisplay.value = URL.createObjectURL(file.value.files[0])
  fileData.value = file.value.files[0]
}

// CREATE POST
const createPost = async () => {
  let dataOut = null
  let errorOut = null

  isLoading.value = true
  // if we have a file we want to upload it to supabase storage
  if (fileData.value) {
    const { data, error } = await client.storage
      .from('threads-clone-files-')
      .upload(`${uuidv4()}.jpg`, fileData.value)

    dataOut = data
    errorOut = error
  }
  // if there is an error we stop the process and return the error
  if (errorOut) {
    console.log(errorOut)
    return errorOut
  }
  // if there is not an error, set pic to nothing because we can create the post without an image
  // if there is a path we're going to assign that to the picture
  let pic = ''
  if (dataOut) {
    pic = dataOut.path ? dataOut.path : ''
  }

  try {
    await useFetch(`/api/create-post/`, {
      method: 'POST',
      body: {
        userId: user.value.identities[0].user_id,
        name: user.value.identities[0].identity_data.full_name,
        image: user.value.identities[0].identity_data.avatar_url,
        text: text.value,
        picture: pic,
      },
    })
    // get fresh data from the database
    await userStore.getAllPosts()
    // close the menu overlay
    userStore.isMenuOverlay = false

    clearData()
    isLoading.value = false
  } catch (error) {
    console.log(error)
    isLoading.value = false
  }
}
</script>
