<template>
    <blog-post-header :profilePhoto="blog?.author?.profilePhoto" :blogUserName="blog?.author?.name"></blog-post-header>
    <div v-if="isLoading" class="w-1/6 m-auto mt-56">
        <q-spinner-grid color="purple" size="5rem"></q-spinner-grid>

    </div>
    <blog-main-content v-else :blog="blog" :postComments="postComments"></blog-main-content>
    <!-- <div class="bg-teal-500 text-white min-h-[400px] w-[220px] m-auto mt-[300px]">
        <p v-if="error">Something went wrong...{{ error }}</p>
        <p v-if="loading">Loading...</p>
        <p v-else>
            Result {{ result }}
        </p>
        <div></div>
    </div> -->
</template>
<script setup lang="ts">
import BlogPostHeader from '@/components/blogPost/BlogPostHeader.vue';
import BlogMainContent from '@/components/blogPost/BlogMainContent.vue';
import { useRoute } from "vue-router"
import { ref, type Ref, watch, computed, onMounted } from "vue"
import { useQuery } from "@vue/apollo-composable";
import {
    GET_A_BLOG_POST_QUERY,
} from "../graphql/BlogQueries";

const isLoading = ref(true)

const route: any = useRoute()
const newBlogPost: Ref<any> = ref(null)



function getBlogNow() {
    const { result, loading, error } = useQuery(GET_A_BLOG_POST_QUERY, {
        id: route.query.postId
    })

    watch(result, (newVal) => {
        console.log("Blog post: ", newVal)
        newBlogPost.value = newVal
        if (result.value) {
            isLoading.value = false
        }
    })

}
onMounted(() => {
    getBlogNow()

})
watch(route, (val) => {
    getBlogNow()
})






const blog = computed(() => {
    return newBlogPost.value?.blogPost
})

const postComments = computed(() => {
    return newBlogPost.value?.postComments
})

</script>