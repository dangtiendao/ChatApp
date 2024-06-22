<template>
  <div>
    <div v-for="message in messages" :key="message.timestamp">
      <strong>{{ message.user }}</strong>: {{ message.text }}
    </div>
    <input v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type a message..." />
  </div>
</template>

<script setup lang="ts">
import * as signalR from '@microsoft/signalr';
import { onMounted, ref } from 'vue';

const connection = ref();
const messages = ref<any[]>([]);
const newMessage = ref();
const user = ref("User1");
onMounted(() => {
  connection.value = new signalR.HubConnectionBuilder()
    .withUrl('http://localhost:5188/chathub')
    .build();

  connection.value.on('ReceiveMessage', (user: any, message: any) => {
    messages.value.push({ user, text: message, timestamp: new Date() });
  });

  connection.value.start().catch((err: any) => console.error(err));
})
function sendMessage() {
  if (newMessage.value.trim() !== '') {
    connection.value.invoke('SendMessage', user.value, newMessage.value)
      .catch((err: any) => console.error(err));
    newMessage.value = '';
  }
}
</script>
