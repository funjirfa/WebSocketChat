<template>
  <div id="app" :class="{ 'modal-close': isOnline }">
    <div class="sign_out">
      <a href="#" @click="disconnect">Покинуть беседу</a>
    </div>
    <div id="chat">
      <chat-message v-for="(message, index) in messageQueue" :message="message" :key="index">

      </chat-message>
    </div>
    <form action="#" method="post" class="send-message-container" @submit.prevent="send">
      <label for="send-message-tool__text" class="send-message-tool__limit">{{ getAvailableCharacter }}</label>
      <div class="send-message-tool">
        <textarea maxlength="300" 
                  id="send-message-tool__text" 
                  name="outgoing_message" 
                  placeholder="введите сообщение" 
                  v-model="outgoingMessage" 
                  @keyup.17.13="send">
          
        </textarea>
        <button type="submit" title="Отправить сообщение [Ctrl + Enter]" class="send-message-tool__btn-send">

        </button>
      </div>
    </form>

    <chat-sign-in-modal class="sign_in" :warning="warning">
      
    </chat-sign-in-modal>
  </div>
</template>

<script>
  import ChatSignInModal from "./components/ChatSignInModal.vue";
  import ChatMessage from "./components/ChatMessage.vue";

export default {
  name: 'app',
  components: {
    'chat-sign-in-modal': ChatSignInModal,
    'chat-message': ChatMessage
  },
  data() {
    return {
      socket: null,
      username: '',
      warning: '',
      isOnline: false,
      date: undefined,
      messageQueue: [],
      outgoingMessage: ''
    }
  },
  methods: {
    onOpen: function () {
      this.isOnline = true;
    },
    onMessage: function (event) {
      let data = JSON.parse(event.data);
      
      this.setTimeStamp(new Date(data['created']));
      
      if (data.hasOwnProperty('name')) {
        this.receiveUsrMsg(data);
      } else {
        this.receiveSysMsgAction(data.text.split(': '));
      }
    },
    onClose: function () {
      this.warningHandler('Вы покинули беседу');
    },
    onError: function (event) {
      this.warningHandler('Возникли проблемы с соединением');
      console.error(event);
    },
    connect: function () {
      let url = new URL('/ws', 'ws://pm.tada.team');
      url.searchParams.append('name', this.username);

      try {
        this.socket = new WebSocket(url.toString());

        this.socket.onopen = this.onOpen;
        this.socket.onmessage = this.onMessage;
        this.socket.onclose = this.onClose;
        this.socket.onerror = this.onError;
      } catch (error) {
        console.error(error);
        this.warningHandler('Не удалось подключиться к серверу');
      }
    },
    disconnect: function () {
      this.socket.close();
    },
    warningHandler: function (warning) {
      this.warning = warning;
      this.isOnline = false;
      this.socket = null;

      let self = this;
      let timeoutId = setTimeout(function () {
        self.warning = '';
        clearTimeout(timeoutId);
      }, 3000);
    },
    setTimeStamp: function (creatingDate) {
      let setDate = false;
      let setTime = false;
      
      if (this.date === undefined) {
        setDate = true;
        setTime = true;
      } else {
        if (this.date.getDay() !== creatingDate.getDay() ||
            this.date.getMonth() !== creatingDate.getMonth() ||
            this.date.getFullYear() !== creatingDate.getFullYear()) {
          setDate = true;
        }
        if (this.date.getMinutes() !== creatingDate.getMinutes() ||
            this.date.getHours() !== creatingDate.getHours()) {
          this.date = creatingDate;
          setTime = true;
        }
      }
      
      if (setDate || setTime) {
        this.date = creatingDate;

        if (setDate) {
          this.receiveSysMsgDate();
        }
        
        if (setTime) {
          this.receiveSysMsgTime();
        }
      }
    },
    receiveUsrMsg: function (data) {
      this.receive({
        type: 'user',
        mode: data.name === this.username ? 'outgoing' : 'incoming',
        name: data.name === this.username ? 'Вы' : data.name,
        text: data.text
      });
    },
    receiveSysMsgDate: function () {
      const weekdays = ['воскресенье', 'понедельник', 'вторник', 'среда', 'четверг', 'пятница', 'суббота'];
      const months = [
          'января', 'февраля', 'марта', 'апреля', 'мая', 'июня', 
          'июля', 'августа', 'сентября', 'октября', 'ноября', 'декабря'
      ];
      
      this.receive({
        type: 'system',
        text: `${this.date.getDay()} ${months[this.date.getMonth()]} ${this.date.getFullYear()}, ${weekdays[this.date.getDay()]}`
      });
    },
    receiveSysMsgTime: function () {
      this.receive({
        type: 'system',
        text: `${this.date.getHours().toString().padStart(2, '0')}:${this.date.getMinutes().toString().padStart(2, '0')}`
      });
    },
    receiveSysMsgAction: function (options) {
      this.receive({
        type: 'system',
        text: `пользователь [${options[1]}] ${options[0] === 'пришёл' ? 'присоединился к беседе' : 'покинул беседу'}`
      });
    },
    receive: function (message) {
      this.messageQueue.push(message);

      if (this.messageQueue.length > 100) {
        this.messageQueue.shift();
      }

      this.scrollToEnd();
    },
    send: function () {
      if (this.outgoingMessage.length > 0) {
        this.socket.send(JSON.stringify({ text: this.outgoingMessage }));
        this.outgoingMessage = '';
      }
    },
    scrollToEnd: function () {
      let chat = this.$el.querySelector('#chat');
      chat.scrollTop = chat.scrollHeight;
    }
  },
  mounted() {
    this.$on('inputUsernameEvent', function (username) {
      if (username.length > 0) {
        this.username = username;
        this.connect();
      } else {
        this.warningHandler('Не указан псевдоним пользователя');
        return false;
      }
    });
  },
  computed: {
    getAvailableCharacter: function () {
      return `Сообщение [ ${300 - this.outgoingMessage.length} / 300 ]:`;
    }
  }
}
</script>

<style>
  @import "~normalize.css";

  html,
  body {
    width: 100%;
    height: 100%;
  }

  body {
    background: #536c67 url("../public/images/background-template.png") 50% 50% repeat;
  }

  #app {
    position: relative;
    display: grid;
    grid-template-columns: 1fr 800px 1fr;
    grid-template-rows: auto 1fr auto;
    grid-template-areas: ". sign_out ."
                         ". conversation ."
                         ". sender .";
    width: 100%;
    height: 100%;
  }

  .sign_out {
    margin-top: 15px;
    grid-area: sign_out;
    padding: 10px 0 10px;
    text-align: center;
    background: rgba(40, 46, 51, 0.75);
  }
  
  .sign_out a {
    font-family: 'Roboto', sans-serif;
    font-size: 14px;
    font-weight: 100;
    text-decoration: none;
    outline: none;
  }

  .sign_out a:link,
  .sign_out a:visited,
  .sign_out a:active {
    color: #f6a321;
  }

  .sign_out a:hover {
    color: #b4a28a;
  }

  #chat {
    grid-area: conversation;
    background: rgba(40, 46, 51, 0.75);
    overflow-y: auto;
  }

  .send-message-container {
    margin-bottom: 15px;
    grid-area: sender;
    padding: 10px 0 10px;
    background: rgba(40, 46, 51, 0.75);
  }
  
  .send-message-tool__limit {
    padding: 15px 0 7px;
    display: block;
    width: 100%;
    font-family: 'Roboto', sans-serif;
    font-size: 14px;
    font-weight: 100;
    text-indent: 15px;
    color: #f6a321;
  }

  .send-message-tool {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  #send-message-tool__text {
    padding: 10px;
    display: block;
    width: 650px;
    height: 50px;
    border: none;
    border-radius: 5px;
    outline: none;
    font-family: 'Roboto', sans-serif;
    font-size: 16px;
    font-weight: 300;
    color: #73d5ff;
    resize: none;
    box-shadow: 0 0 2px -1px #73d5ff;
    background: rgba(40, 46, 51, 0.75);
  }

  .send-message-tool__btn-send {
    position: relative;
    margin-left: 15px;
    display: block;
    width: 50px;
    height: 50px;
    overflow: hidden;
    border-width: 0;
    border-radius: 50%;
    box-shadow: 0 0 2px -1px rgba(251, 232, 206, 0.6);
    background: url('../public/images/btn-send.png') 50% 50% no-repeat;
    background-size: 50% 50%;
    transition: background-color 0.1s;
    cursor: pointer;
  }

  .send-message-tool__btn-send:focus {
    outline: none;
  }

  .send-message-tool__btn-send:before {
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

  .send-message-tool__btn-send:active:before {
    width: 120%;
    padding-top: 120%;
    transition: all 0.1s ease-out;
  }

  .sign_in {
    position: fixed;
    left: 0;
    bottom: 0;
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    grid-template-rows: 1fr auto 1fr;
    grid-template-areas: ". . ."
                         ". sign_in ."
                         ". . .";
    width: 100%;
    height: 100%;
    background: rgba(17, 26, 33, 0.9);
    transition: all 1.75s cubic-bezier(0.86, 0, 0.07, 1);
    z-index: 10;
  }

  #app.modal-close .sign_in {
    bottom: 100%;
  }
</style>
