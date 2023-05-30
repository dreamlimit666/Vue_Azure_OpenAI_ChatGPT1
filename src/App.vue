<template>
  <header class="header">
    <h1>Nick_AI1號-GPT3.5-turbo</h1>
  </header>
  <section class="content" ref="msgContainer">
    <ul id="chat-area" v-for="(message, index) in chatMessages" :key="index">
      <li v-if="message.role == 'Bot'">
        <div class="balloon">
          <img class="img-circle" src="openai.png" alt="image" />
          <p class="talk">{{ message.msg }}</p>
        </div>
      </li>
      <li v-else>
        <div class="balloon balloon-r">
          <p class="talk talk-r">{{ message.msg }}</p>
        </div>
      </li>
    </ul>
  </section>
  <footer class="footer">
    <input
      id="msg-send"
      class="msg-input"
      placeholder="請輸入訊息"
      v-model="prompt"
      v-on:keyup.enter="sendMessage()"
    >
    <button class="btn-submit" type="button" @click="sendMessage()">
      傳送
    </button>
  </footer>
</template>

<script>
import axios from 'axios';
import { Toast } from './utils/helpers';

export default {
  name: 'App',
  data() {
    return {
      selectedOption: 'option1',
      openaiEndpoint: process.env.VUE_APP_OPENAI_ENDPOINT,
      openaiModelDeploymentName: process.env.VUE_APP_OPENAI_MODEL_DEPLOYMENT_NAME,
      openaiKey: process.env.VUE_APP_OPENAI_KEY,
      prompt: '',
      chatMessages: [],
      botMessageArrayIndex: 0,
    };
  mounted() {
    this.chatMessages.push({
      msg: '您好，我是 雲端中心 的 AI1號，有任何問題都可以問我 ^_^',
      role: 'Bot',
    });
  },
  watch: {
    chatMessages: {
      handler() {
        window.scrollTo({
          left: 0,
          top: document.body.scrollHeight,
          behavior: 'smooth',
        });
      },
      deep: true,
    },
  },
  methods: {
    sendMessage() {
      if (this.prompt !== '') {
        this.showUserMessage();
        this.postOpenAI(this.prompt);
        this.prompt = '';
      } else {
        Toast.fire({
          icon: 'error',
          title: '請輸入訊息 !',
        });
      }
    },
    showUserMessage() {
      this.chatMessages.push({
        msg: this.prompt,
        role: 'User',
      });
    },
    showBotMessage(botMessage) {
      this.chatMessages.push({
        msg: botMessage,
        role: 'Bot',
      });
      this.botMessageArrayIndex = this.chatMessages.length - 1;
    },
    async postOpenAI(userPrompt = '') {
      this.showBotMessage('思考中 ~');
      const reqBody = {
        messages: [
          {
            role: 'system',
            content: '你是愛酷智能公司的雲端銷售專家, 專精各種雲端產品, 能回答出各廠牌雲端相關問題, 能銷售雲端產品與硬體相關設備, 也有銷售微軟筆記型電腦(無門市)與各式商用軟體, 但無法提供免費服務, 對於服務與商品折扣請來電愛酷智能雲端中心 電話：02 2711 2900, 並且在回答問題時能找出商機趁機銷售雲端產品, 或協助雲地整合與回答任何資訊問題, 個性和善有禮貌, 喜歡聊天, 會從聊天中掌握商機並反問需求, 只回答雲端相關與資訊類相關問題, 愛酷智能相關資訊：老闆 林庭箴, 雲端總監 威爾森, 公司地址：臺北市松山區敦化南路1段2號5樓, 擁有數十個取得專業認證的技術團隊',
          },
          {
            role: 'user',
            content: userPrompt,
          },
          {
            role: 'assistant',
            content: '',
          },
        ],
        temperature: 0.7,
        top_p: 0.95,
        frequency_penalty: 0,
        presence_penalty: 0,
        max_tokens: 800,
        stop: null,
      };
      try {
        const res = await axios.post(
          'https://nicklab.openai.azure.com/openai/deployments/NickLab/chat/completions?api-version=2023-03-15-preview',
          JSON.stringify(reqBody),
          {
            headers: {
              'content-type': 'application/json',
              'api-key': '7e258da135fd492d9a14e7fcbd6f515c',
            },
          },
        );
        this.chatMessages[this.botMessageArrayIndex].msg = res.data.choices[0].message.content;
      } catch (error) {
        console.log(error.response.data.error);
      }
    },
  },
};
</script>

<style scoped>

* {
  margin: 0;
  padding: 0;
}
.header {
  top: 0;
  position: sticky;
  z-index: 1;
  width: 100%;
  height: 50px;
  font-size: 10px;
  text-align: center;
  line-height: 50px;
  background-color: rgb(30, 144, 255);
  color: white;
}

.content {
  width: 100%;
  min-height: calc(100vh - 100px);
  margin-bottom: 50px;
  background-color: lightgray;
}

#chat-area {
  padding: 10px;
  list-style: none;
}

.balloon {
  margin: 20px 0;
  display: flex;
  align-items: flex-start;
}
.balloon-r {
  margin-right: 15px;
  justify-content: flex-end;
}

.img-circle {
  width: 50px;
  height: 50px;
  margin: 0 15px;
  border-radius: 25px;
  background-color: white;
}

.talk {
  max-width: 500px;
  padding: 10px;
  border-radius: 10px;
  background: white;
}
.talk-r {
  background-color: skyblue;
}

.footer {
  position: fixed;
  z-index: 1;
  bottom: 0;
  width: 100%;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: white;
}

.msg-input {
  width: 70%;
  margin-right: 10px;
  padding: 5px 15px;
  border: 1px solid gray;
  border-radius: 25px;
  background-color: whitesmoke;
}

.btn-submit {
  padding: 6px;
  border: none;
  border-radius: 5px;
  background-color: deepskyblue;
  color: white;
}
</style>
