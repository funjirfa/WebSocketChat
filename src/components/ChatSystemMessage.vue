<template>
  <div class="system-message">
    <slot name="text">
      
    </slot>
  </div>
</template>

<script>
  export default {
    name: "ChatSystemMessage"
  }
</script>

<style>
  .system-message {
    margin: 15px 0;
    color: #ffffff;
    font-family: 'Roboto', sans-serif;
    font-size: 14px;
    font-weight: 100;
    text-align: center;
  }
</style>