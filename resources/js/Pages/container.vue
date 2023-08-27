<script>
import AppLayout from '@/Layouts/AppLayout.vue';
import messageContainer from './Chat/messageContainer.vue';
import inputMessage from './Chat/inputMessage.vue';
import axios from 'axios';
import { ref, watch } from 'vue';
import ChatRoomSelection from './Chat/chatRoomSelection.vue';


export default {
    setup() {
        let chatRooms = ref([]);
        let currentRoom = ref([]);
        let messages = ref([]);

        const getMessages = () => {
            axios.get('/chat/room/' + currentRoom.value.id + '/messages').then(res => {
                messages.value = res.data
            }).catch(err => {
                console.log(err);
            })
        }
        const getRooms = () => {
            axios.get('/chat/rooms').then(res => {
                chatRooms.value = res.data;
                setRoom(res.data[0]);
            }).catch(err => {
                console.log(err);
            });
        };
        const setRoom = (room) => {
            currentRoom.value = room;
            console.log(room);
            getMessages()
        };
        watch(currentRoom, (val, oldVal) => {
            if(oldVal.id) {
                disconnect(oldVal)
            }
            connect()
        })
        const connect = () => {
            if(currentRoom.value.id) {
                let vm = this;
                getMessages()
                window.Echo.private("chat." + currentRoom.value.id).listen('.message.new', e => {
                    vm.getMessages()
                })
            }
        }
        const disconnect = (room) => {
            window.Echo.leave('chat.' + room.id)
        }
        getRooms();
        return {
            chatRooms,
            currentRoom,
            messages,
            setRoom,
            connect
        };
    },
    components: { messageContainer, inputMessage, AppLayout, ChatRoomSelection }
}
</script>

<template>
    <AppLayout title="Dashboard">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <ChatRoomSelection v-if="currentRoom.id" :rooms="chatRooms" :currentRoom="currentRoom" v-on:roomchanged="setRoom($event)"></ChatRoomSelection>
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <messageContainer :messages="messages"></messageContainer>
                    <inputMessage :room="currentRoom" v-on:messagesent="getMessages()"></inputMessage>
                </div>
            </div>
        </div>
    </AppLayout>
</template>
