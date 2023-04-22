<script setup>
// prop : 부모가 자식에게 전달해주는 데이터
import { ref, computed } from 'vue';
import dayjs from 'dayjs';
import 'dayjs/locale/ko'

dayjs.locale('ko')

const props = defineProps({
  messages: [Object],
  currentUser : String
})

function formatDate(date) {
    return dayjs(date).format('M월 D일 A h:mm');
}

</script>

<template>
  <template v-for="(msg, i) of props.messages" :key="msg.id">
    <div :class="{ my: msg.username === currentUser }" class="msg">
      <div v-show="messages[i].username != messages[i - 1]?.username" class="name">{{ msg.username }}</div>
      <div class="chat" v-if="!msg.enter">
        <div class="text" @mouseenter="msg.isHover = true" @mouseleave="msg.isHover = false">{{ msg.content }}</div> 
        <div class="time" v-show="msg.isHover">{{ formatDate(msg.time) }}</div>
      </div>
      <div class="enter" v-if="msg.enter">{{ msg.enter }}</div>
    </div>
  </template>
</template>


<style scoped>

.msg {
  display: flex;
  flex-direction: column;
  line-height: 170%; 
  margin-bottom: 12px;
  margin-left: 32px;
  margin-right: 32px;
}

.name {
  color: #262f37;
  margin-left: 6px;
}

.chat {
  display: flex;
  align-items: center;
  gap: 12px;
}

.text {
  background-color: #5e6e82;
  color: #f8ffff;
  border-radius: 12px;
  padding: 5px 10px 5px 10px;
  max-width: 300px;
  word-break: break-all;
}

.time {
  font-size: 16px;
  background-color: black;
  color: #f8ffff;
  text-align: center;
  padding: 5px;
  align-self: center;
  width: 140px;
  border-radius: 10px;
}

.my {
  align-items: flex-end;
}

.my .name {
  margin-right: 3px;
}

.my .chat {
  flex-direction: row-reverse;
}

.my .text {
  background-color: #5a9dd2;
}

.enter {
  margin: 15px 0 10px 0;
  text-align: center;
  font-size: 22px;
}

</style>
