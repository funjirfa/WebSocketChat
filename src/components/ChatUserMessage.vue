<template>
  <div class="user-message" :class="{'user-message_outgoing': isModeOutgoing}">
    <div class="user-message__user" :class="{'user-message__user_outgoing': isModeOutgoing}">
      <slot name="user">
        
      </slot>
    </div>
    <div class="user-message__tail" :class="{'user-message__tail_outgoing': isModeOutgoing}"></div>
    <div class="user-message__text" :class="{'user-message__text_outgoing': isModeOutgoing}">
      <slot name="text">
        
      </slot>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ChatUserMessage",
    props: {
      mode: {
        type: String,
        default: 'incoming'
      }
    },
    computed: {
      isModeOutgoing: function () {
        return this.mode === 'outgoing';
      }
    }
  }
</script>

<style>
  .user-message {
    position: relative;
    margin: 15px 0;
  }

  .user-message_outgoing {
    text-align: right;
  }
  
  .user-message__user {
    margin-left: 10px;
    padding-bottom: 7px;
    height: 20px;
    line-height: 20px;
    color: #2bb19a;
    font-family: 'Roboto', sans-serif;
    font-size: 18px;
    font-weight: 300;
    font-style: italic;
  }

  .user-message__user_outgoing {
    margin-right: 10px;
    color: #e94c40;
    text-align: right;
  }
  
  .user-message__tail {
    position: absolute;
    left: 10px;
    top: 27px;
    width: 20px;
    height: 10px;
    background: #73d5ff;
    border-radius: 3px;
    transform: skew(65deg);
    z-index: 1;
  }

  .user-message__tail_outgoing {
    left: auto;
    right: 10px;
    background: #fbcc83;
    transform: skew(-65deg);
  }
  
  .user-message__text {
    position: relative;
    margin-left: 15px;
    display: inline-block;
    max-width: 650px;
    padding: 8px 12px;
    background: linear-gradient(to bottom, #73d5ff 0%, #00b4ff 100%);
    border-radius: 0 5px 5px;
    box-shadow: 2px 2px 3px -1px #000000;
    font-family: 'Roboto', sans-serif;
    font-size: 18px;
    font-weight: 500;
    text-shadow: 1px 1px 1px #f9f9f9;
    z-index: 2;
  }
  
  .user-message__text_outgoing {
    margin-right: 15px;
    border-radius: 5px 0 5px 5px;
    background: linear-gradient(to bottom, #fbcc83 0%, #f6a321 100%);
    box-shadow: -2px 2px 3px -1px #000000;
  }  
</style>