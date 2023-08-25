<template>
    <div class="h-96 w-full">
        <div class="h-full p-2 flex flex-col-reverse overflow-scroll">
            <div v-for="(message, index) in messages" :key="index">
                <MessageItem :message="message"></MessageItem>
            </div>
        </div>
    </div>
</template>

<script>
import MessageItem from './MessageItem.vue';

    export default {
    props: ['messages'],
    components: { MessageItem }
}
</script>