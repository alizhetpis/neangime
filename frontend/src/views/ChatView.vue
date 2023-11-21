<template>
    <div class="flex justify-center py-4">
        <div class="max-w-7xl mx-auto grid grid-cols-4 gap-4">
            <!-- Chat List (Narrower Column) -->
            <div class="md:col-span-1 bg-white border border-gray-200 rounded-lg p-4 text-center overflow-auto max-h-[90vh]">
                <div class="space-y-2">
                    <div 
                        class="flex items-center space-x-3 p-2 rounded-lg cursor-pointer hover:bg-gray-100"
                        v-for="conversation in conversations"
                        :key="conversation.id"
                        @click="setActiveConversation(conversation.id)"
                        :class="{ 'bg-gray-200': activeConversation.id === conversation.id }"
                    >
                        <!-- Avatar -->
                        <img :src="conversation.users[0].get_avatar" class="w-10 h-10 rounded-full object-cover">
        
                        <!-- Name and Timestamp -->
                        <div class="flex flex-col">
                            <span class="font-semibold">{{ conversation.users[0].name }}</span>
                            <span class="text-gray-500 text-sm">{{ conversation.modified_at_formatted }} ago</span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Main Chat Area (Wider Column) -->
            <div class="col-span-3 space-y-4">
                <div class="bg-white border border-gray-200 rounded-lg flex flex-col flex-grow p-4">
                    <template v-for="message in activeConversation.messages" v-bind:key="message.id">
                        <div 
                            class="flex w-full mt-2 space-x-3 max-w-md ml-auto justify-end"
                            v-if="message.created_by.id == userStore.user.id"
                        >
                            <div>
                                <div class="bg-blue-600 text-white p-3 rounded-l-lg rounded-br-lg">
                                    <p class="text-sm">{{ message.body }}</p>
                                </div>
                                <span class="text-xs text-gray-500 leading-none">{{ message.created_at_formatted }} ago</span>
                            </div>
                            <div class="flex-shrink-0 h-10 w-10 rounded-full bg-gray-300">
                                <img :src="message.created_by.get_avatar" class="w-[40px] rounded-full">
                            </div>
                        </div>

                        <div 
                            class="flex w-full mt-2 space-x-3 max-w-md"
                            v-else
                        >
                            <div class="flex-shrink-0 h-10 w-10 rounded-full bg-gray-300">
                                <img :src="message.created_by.get_avatar" class="w-[40px] rounded-full">
                            </div>
                            <div>
                                <div class="bg-gray-300 p-3 rounded-r-lg rounded-bl-lg">
                                    <p class="text-sm">{{ message.body }}</p>
                                </div>
                                <span class="text-xs text-gray-500 leading-none">{{ message.created_at_formatted }} ago</span>
                            </div>
                        </div>
                    </template>
                </div>
                <div class="bg-white border border-gray-200 rounded-lg">
                    <form v-on:submit.prevent="submitForm">
                        <div class="p-4">  
                            <textarea v-model="body" class="p-4 w-full bg-gray-100 rounded-lg" placeholder="What do you want to say?"></textarea>
                        </div>
                        <div class="p-4 border-t border-gray-100 flex justify-between">
                            <button class="inline-block py-4 px-6 bg-purple-600 text-white rounded-lg">Send</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
import axios from 'axios'
import { useUserStore } from '@/stores/user'

export default {
    name: 'chat',

    setup() {
        const userStore = useUserStore()

        return {
            userStore
        }
    },

    data() {
        return {
            conversations: [],
            activeConversation: {},
            body: '',
            hoveredConversation: null 
        }
    },

    mounted() {
        this.getConversations()
    },
    
    methods: {
        setActiveConversation(id) {
            console.log('setActiveConversation', id)

            this.activeConversation = id
            this.getMessages()
        },
        getConversations() {
            console.log('getConversations')

            axios
                .get('/api/chat/')
                .then(response => {
                    console.log(response.data)

                    this.conversations = response.data

                    if (this.conversations.length) {
                        this.activeConversation = this.conversations[0].id
                    }

                    this.getMessages()
                })
                .catch(error => {
                    console.log(error)
                })
        },

        getMessages() {
            console.log('getMessages')

            axios
                .get(`/api/chat/${this.activeConversation}/`)
                .then(response => {
                    console.log(response.data)

                    this.activeConversation = response.data
                })
                .catch(error => {
                    console.log(error)
                })
        },

        submitForm() {
            console.log('submitForm', this.body)

            axios
                .post(`/api/chat/${this.activeConversation.id}/send/`, {
                    body: this.body
                })
                .then(response => {
                    console.log(response.data)

                    this.activeConversation.messages.push(response.data)
                })
                .catch(error => {
                    console.log(error)
                })
        }
    }
}
</script>

<style>
/* Add any additional styling you need for the hover effect here */
.bg-gray-100 {
  background-color: #f7f7f7; /* Example color for the hover background */
}
</style>