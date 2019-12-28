<template>
  <div>
    <form action="#"
          method="post"
          class="sign_in__content"
          @submit.prevent="setUsername">
      <label for="sign_in__username" class="sign_in__welcome">
        Чтобы присоединиться к беседе, представьтесь...
      </label>
      <div class="sign_in__controls">
        <input autofocus
               size="20"
               maxlength="20"
               name="username"
               placeholder="ваш псевдоним"
               id="sign_in__username"
               v-model="username">
        <button
          class="sign_in__btn-entry"
          type="submit">
          <span>Начать общение</span>
        </button>
      </div>
      <transition name="fade">
        <div  class="sign_in__message" v-show="isWarningOccurred">{{ warning }}</div>
      </transition>
    </form>
  </div>
</template>

<script>
  export default {
    name: "ChatSignInModal",
    props: {
      warning: {
        type: String,
        default: ''
      }
    },
    data() {
      return {
        username: ''
      }
    },
    methods: {
      setUsername: function () {
        this.$parent.$emit('inputUsernameEvent', this.username);
      }
    },
    computed: {
      isWarningOccurred: function () {
        return this.warning.length > 0;
      }
    }
  }
</script>

<style>
  .sign_in__content {
    grid-area: sign_in;
  }

  .sign_in__welcome {
    display: block;
    width: 800px;
    font-family: 'Exo 2', sans-serif;
    font-size: 30px;
    font-weight: 700;
    color: #8796a2;
    text-align: center;
    text-shadow: #443217 1px 1px 2px;
  }

  .sign_in__controls {
    margin-top: 35px;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
  }

  #sign_in__username {
    display: block;
    padding: 10px 15px;
    color: #7e91a2;
    font-family: 'Exo 2', sans-serif;
    font-size: 25px;
    font-weight: 300;
    text-align: center;
    border-width: 0;
    box-shadow: 0 1px 1px -1px #b4a795;
    outline: none;
    background-color: transparent;
  }

  .sign_in__btn-entry {
    position: relative;
    margin-left: 35px;
    display: block;
    overflow: hidden;
    border-width: 0;
    border-radius: 2px;
    outline: none;
    box-shadow: 0 1px 2px rgba(251, 232, 206, 0.6);
    background-color: #3baa36;
    color: #d1ead0;
    transition: background-color 0.1s;
    cursor: pointer;
    font-family: 'Exo 2', sans-serif;
    font-size: 20px;
    font-weight: 700;
  }

  .sign_in__btn-entry span {
    padding: 12px 24px;
    display: block;
  }

  .sign_in__btn-entry:before {
    content: "";
    position: absolute;
    padding-top: 0;
    top: 50%;
    left: 50%;
    width: 0;
    border-radius: 100%;
    background-color: rgba(236, 240, 241, 0.3);
    transform: translate(-50%, -50%);
  }

  .sign_in__btn-entry:active:before {
    width: 120%;
    padding-top: 120%;
    transition: all 0.1s ease-out;
  }

  .sign_in__message {
    margin-top: 25px;
    font-family: 'Exo 2', sans-serif;
    font-size: 15px;
    font-weight: 300;
    text-align: center;
    color: #fade1f;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity 1.5s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>