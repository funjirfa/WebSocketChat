<template>
  <chat-system-message v-if="message.type === 'system'">
    <template v-slot:text>{{ message.text }}</template>
  </chat-system-message>
  <chat-user-message v-else :mode="message.mode">
    <template v-slot:user>~ {{ message.name }}</template>
    <template v-slot:text>{{ message.text }}</template>
  </chat-user-message>
</template>

<script>
  import ChatSystemMessage from "./ChatSystemMessage.vue";
  import ChatUserMessage from "./ChatUserMessage.vue";

  export default {
    name: "ChatMessage",
    components: {
      'chat-system-message': ChatSystemMessage,
      'chat-user-message': ChatUserMessage
    },
    props: {
      message: {
        type: Object
      }
    }
  }
</script>

<style>

</style>