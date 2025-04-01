<template>
    <div id="app">
        <ChatBot />
    </div>
</template>

<script>
import ChatBot from './components/ChatBot.vue';

export default {
    components: {
        ChatBot,
    },
};
</script>

<style>
#app {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #f1f1f1;
}
</style>
