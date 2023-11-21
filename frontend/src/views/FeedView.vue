<template>
    <div class="flex justify-center min-h-screen">
      <div class="w-full max-w-4xl p-4">
        <!-- FeedForm -->
        <div class="bg-white border border-gray-200 rounded-lg mb-4">
          <FeedForm v-bind:user="null" v-bind:posts="posts" />
        </div>
  
        <!-- Posts -->
        <div 
          class="mb-6 bg-white border border-gray-200 rounded-lg p-4" 
          v-for="post in posts"
          v-bind:key="post.id"
        >
          <FeedItem v-bind:post="post" v-on:deletePost="deletePost" />
        </div>
      </div>
    </div>
  </template>
  
  


<script>
import axios from 'axios'
import PeopleYouMayKnow from '../components/PeopleYouMayKnow.vue'
import Trends from '../components/Trends.vue'
import FeedItem from '../components/FeedItem.vue'
import FeedForm from '../components/FeedForm.vue'

export default {
    name: 'FeedView',

    components: {
        PeopleYouMayKnow,
        Trends,
        FeedItem,
        FeedForm
    },

    data() {
        return {
            posts: [],
            body: '',
        }
    },

    mounted() {
        this.getFeed()
    },

    methods: {
        getFeed() {
            axios
                .get('/api/posts/')
                .then(response => {
                    console.log('data', response.data)

                    this.posts = response.data
                })
                .catch(error => {
                    console.log('error', error)
                })
        },

        deletePost(id) {
            this.posts = this.posts.filter(post => post.id !== id)
        },
    }
}
</script>