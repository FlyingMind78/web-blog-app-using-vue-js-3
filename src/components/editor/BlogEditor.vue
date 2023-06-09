<template>
  <div class=" z-50 fixed top-2 left-0 w-full h-full  bg-white mb-[10rem] overflow-y-auto overflow-x-hidden">

    <div class=" relative w-[700px] m-auto p-6 border-2 border-teal-400 rounded-md my-10">
      <h1
        class="text-center text-[42px] text-transparent bg-gradient-to-r from-[#c71e57] via-[#da3418] to-[#2c6bb1] bg-clip-text font-bold ">
        Editor
      </h1>
      <div @click="emit('close-editor-dialog')"
        class=" cursor-pointer absolute top-2 right-[3rem] text-2xl text-red-800 font-medium">x</div>
      <div class="my-5">

        <q-form>
          <div class="text-teal-900 text-lg text-teal-600 font-semibold mt-1 ">Title</div>
          <q-input name="title" placeholder="Enter a title of your blog" v-model="blogTitle"></q-input>
          <div class="text-teal-900 text-lg text-teal-600 font-semibold  mt-1">Brief </div>
          <q-input name="title" placeholder="Enter a brief about your blog" v-model="blogBrief"></q-input>
          <div class="text-teal-900 text-lg text-teal-600 font-semibold  mt-1">Tags </div>
          <q-input name="title" placeholder="Enter some tags seperated by a space " v-model="blogTags"
            hint="Give a space for each tag"></q-input>
          <div class="text-teal-900 text-lg text-teal-600 font-semibold  mt-1">Reading time </div>
          <q-input name="title" placeholder="Enter your expected reading time a reader might take "
            v-model="blogReadingTime" hint="Expected reading time"></q-input>
          <div v-if="editorMode === 'edit'">
            <div class="text-teal-900 text-lg text-teal-600 font-semibold mt-3 mb-2">Current cover photo</div>
            <img :src="oldCoverImage" alt="old cover image">
          </div>

          <div class="text-teal-900 text-lg text-teal-600 font-semibold mt-3 mb-2">Upload a Cover photo</div>
          <q-file accept="image/*" color="purple-12" v-model="coverImageFile" label="Upload a cover photo">
            <template v-slot:prepend>
              <q-icon name="attach_file" />
            </template>
            <template v-slot:append>
              <q-icon name="close" @click.stop.prevent="coverImageFile = ''" class="cursor-pointer" />
            </template>
          </q-file>
        </q-form>
      </div>
      <!-- <div class="h-[300px] my-4 bg-teal"> -->
      <div class="text-teal-900 text-lg text-cyan-600 font-semibold  my-4 uppercase">
        Write your blog content here
      </div>


      <div>

        <quill-editor ref="editor" :modules="modules" class="min-h-[350px]  border-2 border-gray-400 rounded-md my-4"
          v-model:content="editorBlogContent" toolbar="full" contentType="html"></quill-editor>
      </div>



      <q-btn v-if="editorMode === 'create'" class="my-5" @click="createBlogPostHandler" label="create post"
        color="primary"></q-btn>
      <q-btn v-else-if="editorMode === 'edit'" class="my-5" @click="updatePostHandler" label="Update post"
        color="orange"></q-btn>
      <q-btn @click="emit('close-editor-dialog')" label="cancel" class="mx-3" color="teal"></q-btn>
    </div>


  </div>
</template>
<script setup lang="ts">

import { QuillEditor } from '@vueup/vue-quill'

import { useBlogStore } from "../../stores/blog";
import { ref, type Ref } from "vue"
import BlotFormatter from 'quill-blot-formatter'

import { v4 as uuidv4 } from 'uuid';
import * as Realm from "realm-web";




const { allUserBlogs, editorMode, postId }: any = defineProps({
  allUserBlogs: {
    type: Array,
    required: false
  },
  editorMode: {
    type: String,
    required: true
  },
  postId: {
    type: String,
    required: false
  }
})
const { title, brief, tags, coverImage, content, readingTime, createdAt }: any = allUserBlogs ? allUserBlogs?.find((blog: any) => blog?.postId === postId) :

  {
    title: "",
    brief: "",
    tags: "",
    coverImage: "",
    content: "",
    readingTime: "",
    createdAt: ""
  }

const { BSON } = Realm
const emit = defineEmits(["close-editor-dialog"])

const editor: any = ref(null);

const blogTitle: Ref<string> = ref(title || "")
const blogBrief: Ref<string> = ref(brief || "")
const blogTags: Ref<string> = ref(tags ? tags?.join("|") : "")
const oldCoverImage: Ref<string> = ref(coverImage)
const coverImageFile: Ref<any> = ref(null)
const editorBlogContent: Ref<string> = ref(content || "")
const blogReadingTime: Ref<string> = ref(readingTime || "")


const { createABlogPost, updateABlogPost } = useBlogStore()

const modules: any = {
  name: 'blotFormatter',
  module: BlotFormatter,
  options: null
}

function resetBlogEditor() {
  blogTitle.value = ""
  blogBrief.value = ""
  blogTags.value = ""
  blogReadingTime.value = ""
  coverImageFile.value = ""
  editor.value.setContents(null)
}

function updatePostHandler() {
  console.log("updating post")
  const updateBlogData: any = {
    "title": blogTitle.value,
    "brief": blogBrief.value,
    "content": editorBlogContent.value,
    "updatedAt": new Date().toISOString(),
    "tags": blogTags?.value?.toUpperCase().split(" "),
    "readingTime": blogReadingTime.value,
    "postId": postId,
    "author": { link: new BSON.ObjectID(localStorage.getItem("u_obj_id") || "") },
    "createdAt": createdAt,
    "coverImage": oldCoverImage.value
  }
  if (coverImageFile.value !== null) {
    const reader: any = new FileReader()
    reader.readAsDataURL(coverImageFile.value)
    reader.onload = () => {
      const base64CoverImage = reader.result;
      updateBlogData.coverImage = base64CoverImage;
      updateABlogPost(postId, updateBlogData)
      resetBlogEditor()
    }
  } else {
    updateABlogPost(postId, updateBlogData)
    resetBlogEditor()
  }


}

function createBlogPostHandler() {
  const blogData: any = {
    "title": blogTitle.value,
    "brief": blogBrief.value,
    "content": editorBlogContent.value,
    "createdAt": new Date().toISOString(),
    "tags": blogTags.value.toUpperCase().split("|"),
    "postId": uuidv4(),
    "readingTime": blogReadingTime.value,
    "author": { link: new BSON.ObjectID(localStorage.getItem("u_obj_id") || "") }
  }

  if (coverImageFile.value !== null) {
    const reader: any = new FileReader()
    reader.readAsDataURL(coverImageFile.value)
    reader.onload = () => {
      const base64CoverImage = reader.result;
      blogData.coverImage = base64CoverImage;
      console.log("Blog data", blogData)
      createABlogPost(blogData)
      resetBlogEditor();
    }
  }




}


</script>