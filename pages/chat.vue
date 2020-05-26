<template>
  <v-container class="chat-page">
    <v-row justify="center">
      <v-col>
        <center>
          <div v-if="room" class="display-1">Room: {{ room.name }}</div>
        </center>
      </v-col>
    </v-row>
    <v-row v-for="message in messages" :key="message.id" justify="center">
      <v-col
        :class="user.id == message.author ? 'my-message' : 'other-message'"
        cols="12"
        md="8"
        xl="5"
      >
        <v-avatar v-if="user.id != message.author">
          <img :src="message.authorIcon" />
        </v-avatar>
        <v-card
          class="message-card"
          :color="user.id == message.author ? '#AED581' : 'white'"
        >
          <v-card-title v-text="message.text"></v-card-title>
        </v-card>
        <v-avatar v-if="user.id == message.author">
          <img :src="message.authorIcon" />
        </v-avatar>
      </v-col>
    </v-row>
    <v-app-bar app bottom fixed height="80">
      <v-container style="padding: 0; margine: 0;">
        <v-row justify="center" align="center">
          <v-col cols="6">
            <v-text-field
              v-model="text"
              solo
              hide-details
              placeholder="message"
              @keyup.enter="onPost"
            ></v-text-field>
          </v-col>
          <v-col cols="1">
            <v-btn fab elevation="3" color="primary" @click="onPost">
              <v-icon>mdi-send</v-icon>
            </v-btn>
          </v-col>
          <v-col cols="1">
            <v-btn fab elevation="3" color="primary" @click="onPostSound(1)">
              👏
            </v-btn>
          </v-col>
          <v-col cols="1">
            <v-btn fab elevation="3" color="primary" @click="onPostSound(2)">
              ｳｵｫｫ!!
            </v-btn>
          </v-col>
          <v-col cols="1">
            <v-btn fab elevation="3" color="primary" @click="onPostSound(3)">
              😂
            </v-btn>
          </v-col>
          <v-col cols="1">
            <v-btn fab elevation="3" color="primary" @click="onPostSound(4)">
              ェ(ﾟДﾟ)ェ
            </v-btn>
          </v-col>
          <v-col cols="1">
            <v-btn fab elevation="3" color="primary" @click="onPostSound(5)">
              🤯
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-app-bar>
  </v-container>
</template>

<script>
import io from 'socket.io-client'
import { mapGetters, mapActions } from 'vuex'
import moment from 'moment'

export default {
  data() {
    return {
      text: null,
      socket: null,
      messages: [],
    }
  },
  computed: {
    ...mapGetters(['user', 'room']),
  },
  created() {
    this.socket = io()
    this.socket.emit('join-room', {
      id: this.user.id,
      roomId: this.$route.query.roomId,
    })
    this.socket.on('init-messages', this.initMessages)
    this.socket.on('send-message', this.recieveMessage)
    this.getRoom({ id: this.$route.query.roomId })
  },
  methods: {
    initMessages(messages) {
      this.messages = messages
    },
    recieveMessage(message) {
      if (message.sound) {
        const sound = document.createElement('audio')
        sound.src = message.sound
        sound.play()
      }
      this.messages.push(message)
    },
    onPost() {
      if (this.text) {
        const message = {
          // author: this.user.id,
          // authorIcon: this.user.picture,
          text: this.text,
          roomId: this.$route.query.roomId,
        }
        this.socket.emit('send-message', message)
        this.text = null
      }
    },
    onPostSound(soundNumber) {
      let text = ''
      let sound = ''
      switch (soundNumber) {
        case 1:
          text = '👏'
          sound =
            'https://soundeffect-lab.info/sound/voice/mp3/people/people-performance-cheer2.mp3'
          break

        case 2:
          text = 'Woooooow!!'
          sound =
            'https://soundeffect-lab.info/sound/voice/mp3/people/people-stadium-cheer2.mp3'
          break

        case 3:
          text = '😂'
          sound =
            'https://soundeffect-lab.info/sound/voice/mp3/people/people-studio-laugh-large2.mp3'
          break

        case 4:
          text = 'ェ(ﾟДﾟ)ェ'
          sound =
            'https://soundeffect-lab.info/sound/voice/mp3/people/people-studio-ee1.mp3'
          break

        case 5:
          text = '🤯'
          sound = 'https://soundeffect-lab.info/sound/battle/mp3/bomb1.mp3'
          break

        default:
          break
      }
      const message = {
        text,
        roomId: this.$route.query.roomId,
        sound,
      }
      this.socket.emit('send-message', message)
      this.text = null
    },
    format(timestamp) {
      return moment(timestamp.seconds * 1000).format('YYYY/MM/DD HH:mm:ss')
    },
    ...mapActions(['getRoom']),
  },
}
</script>

<style lang="scss">
.chat-page {
  .my-message {
    display: flex;
    align-items: center;
    .message-card {
      margin-left: auto;
    }
  }
  .other-message {
    display: flex;
    align-items: center;
  }
  .message-card {
    margin-left: 10px;
    margin-right: 10px;
  }
}
</style>
