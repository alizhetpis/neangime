<template>
    <div class="flex justify-center min-h-screen">
        <div class="w-full max-w-4xl p-4">
            <template v-if="notifications.length">
                <div 
                    class="p-4 bg-white border border-gray-200 rounded-lg hover:bg-gray-100 transition duration-300 ease-in-out"
                    v-for="notification in notifications"
                    :key="notification.id"
                >
                    <!-- Notification Content -->
                    <div class="flex items-center space-x-3">
                        <!-- User Avatar -->
                        <img 
                            :src="notification.userAvatarUrl" 
                            alt="User's avatar"
                            class="w-10 h-10 rounded-full"
                        />

                        <div class="flex-grow">
                            <p>{{ notification.body }}</p>
                            <!-- Notification Date -->
                            <div class="text-sm text-gray-500">{{ formatDate(notification.created_at) }}</div>
                            <button class="underline text-sm text-blue-600" @click="readNotification(notification)">Read more</button>
                        </div>
                    </div>
                </div>
            </template>
            <div v-else class="p-4 bg-white border border-gray-200 rounded-lg">
                You don't have any unread notifications!
            </div>
        </div>
    </div>
</template>


<script>

import axios from 'axios'

export default {
    name: 'notifications',

    data() {
        return {
            notifications: []
        }
    },

    mounted() {
        this.getNotifications()
    },

    methods: {
        getNotifications() {
        axios
            .get('/api/notifications/')
            .then(response => {
                console.log(response.data);

                // Assuming each notification has a 'created_at' or 'updated_at' field
                this.notifications = response.data.sort((a, b) => {
                    // Convert the dates to timestamps and compare
                    return new Date(b.created_at) - new Date(a.created_at);
                });
            })
            .catch(error => {
                console.log('Error: ', error);
            });
    },

        async readNotification(notification) {
            console.log('readNotification', notification.id)

            await axios
                .post(`/api/notifications/read/${notification.id}/`)
                .then(response => {
                    console.log(response.data)

                    if (notification.type_of_notification == 'post_like' || notification.type_of_notification == 'post_comment') {
                        this.$router.push({name: 'postview', params: {id: notification.post_id}})
                    } else {
                        this.$router.push({name: 'friends', params: {id: notification.created_for_id}})
                    }
                })
                .catch(error => {
                    console.log('Error: ', error)
                })
        }
    }
}
</script>