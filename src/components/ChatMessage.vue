<script setup>
// prop : 부모가 자식에게 전달해주는 데이터
import { ref } from 'vue';
import dayjs from 'dayjs';
import 'dayjs/locale/ko'

dayjs.locale('ko')

const props = defineProps({
  content: String,
  time: String,
  enter: {
    type: String,
    default: ""
  }
})

const isHover = ref(false);

function formatDate(date) {
    return dayjs(date).format('M월 D일 A h:mm');
}

</script>

<template>
  <div class="chat" v-if="!props.enter">
      <div class="text" @mouseenter="isHover = true" @mouseleave="isHover = false">{{ props.content }}</div>
      <div class="time" v-show="isHover">{{ formatDate(props.time) }}</div>
  </div>
  <div class="enter" v-if="props.enter">{{ props.enter }}</div>
</template>

<style scoped>

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

.my .text {
  background-color: #5a9dd2;
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

</style>
