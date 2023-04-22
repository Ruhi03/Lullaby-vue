<script setup>

import { ref, nextTick } from 'vue'; 
import { io } from 'socket.io-client'
import ChatMessage from './components/ChatMessage.vue';
// import fileInput from './components/fileInput.vue';
// import textInput from './components/textInput.vue';
import InputImage from './components/inputImage.vue';
import InfiniteLoading from "v3-infinite-loading";

const socket = io(`http://${location.hostname}:3000`);

const messages = ref([]);
const images = ref([]);

const scroll = ref();
const input = ref();
const inputFile = ref();
const showDiv = ref(false);

let offset = 0;
let username;



// 메시지가 출력되고 나서 가장 아래로 스크롤 되도록 하는 함수
function scrollTobottom() {
    nextTick(() => {
        if (scroll.value !== null) {
            scroll.value.scrollTo({
                top: scroll.value.scrollHeight,
                left: 0,
                behavior: 'smooth',
            });
        }
    }) 
}

// 사이트가 진입하고 최초로 메시지 로드할 경우
socket.once('first load message', data => {

    // 사용자 이름을 입력 받음
    username = prompt('이름을 입력해주세요');

    // prompt 창에서 입력하지 않고 탈출할 경우
    // null 이나 빈 문자열을 return 함
    // 그럴 경우는 익명으로 닉네임 대체
    if (username === null || username === '') {
        username = '익명';
    }

    // 메시지들을 messages array 에 내부 값만 집어넣음
    messages.value = messages.value.concat(data.data);
    offset += data.data.length
    
    nextTick(() => {
        socket.emit('enter user', username);
    })
});

socket.on('enter user', username => {
    const msg = {
        enter: `${username}님이 입장하셨습니다.`
    }

    messages.value.push(msg);
    scrollTobottom();
});

socket.on('send message', msg => {
    messages.value.push(msg);
    scrollTobottom();
});

function onSend() {

    // Input.value 가 비어있을 때는 undefined을 return 하므로
    // undefined 일 경우 실행되지 않도록
    if ((input.value !== undefined && input.value.trim() !== '') || images.value.length !== 0) {
        const message = {
            username: username,
            content: input.value,
            filePaths: images.value,
            createdAt: String(Date.now())
        };

        socket.emit('send message', message);
        input.value = '';
        images.value = [];
    }
}

const load = $state => {
    try {
        socket.emit('load message', offset);

        // 스크롤 할때 마다 한번씩만 받도록 socket.once 사용
        socket.once('load message', data => {

        // 가져온 메시지의 개수가 10개 이하라면
        // 즉 남은 메시지 개수가 10개 이하
        // 메시지를 다 가져온다면
        if (data.data.length < 10) {

            // 마지막 로드 과정에 스크롤이 이상해 1픽셀 스크롤
            scroll.value.scrollTo(0, 1);
            messages.value.unshift(...data.data);
            $state.complete();
        
        // 메시지가 아직 10개 이상 남아있을 경우
        } else {
            messages.value.unshift(...data.data);
            $state.loaded();
        }
        
        offset += data.data.length;
        });
    } catch (error) {
        $state.error();
    }
};

function readerAndPush(imageFile) {
    const reader = new FileReader()
    reader.onload = (event) => {

        const image = {
            url: event.target.result
        }

        images.value.push(image);
    }
    reader.readAsDataURL(imageFile)
}

function onPaste(event) {
    const clipboardData = event.clipboardData || window.Clipboard;
    const items = clipboardData.items;
    for (let i = 0; i < items.length; i++) {
        if (items[i].type.indexOf("image") !== -1) {
            const imageFile = items[i].getAsFile();
            readerAndPush(imageFile);
        }
    }
}

function selectFile() {
    inputFile.value.click();
}

function handleFileSelect(event) {
    const file = event.target.files[0]
    readerAndPush(file);

}

function handleScroll(event) {
    const scrollHeight = event.target.scrollHeight; // 스크롤 가능한 전체 높이
    const scrollTop = event.target.scrollTop; // 현재 스크롤 위치
    const clientHeight = event.target.clientHeight; // 스크롤 가능한 뷰포트의 높이
    const scrollBottom = scrollHeight - scrollTop - clientHeight;
    // console.log(scrollBottom)

    if (scrollBottom > 1200) {
        showDiv.value = true;

    } else {
        showDiv.value = false;

    }
}

</script>
 
<template>
    <div class="messages" @scroll="handleScroll" ref="scroll">
        <InfiniteLoading @infinite="load" target=".messages" :top="true" :firstload="false">
            <template #complete><div></div></template>
        </InfiniteLoading>
        <ChatMessage :messages="messages" :currentUser="username"></ChatMessage>
    </div>
    <div class="form">
        <svg class="scroll-btn" :style="{ opacity: showDiv ? 1 : 0 }" @click="scrollTobottom" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M256 0a256 256 0 1 0 0 512A256 256 0 1 0 256 0zM376.9 294.6L269.8 394.5c-3.8 3.5-8.7 5.5-13.8 5.5s-10.1-2-13.8-5.5L135.1 294.6c-4.5-4.2-7.1-10.1-7.1-16.3c0-12.3 10-22.3 22.3-22.3l57.7 0 0-96c0-17.7 14.3-32 32-32l32 0c17.7 0 32 14.3 32 32l0 96 57.7 0c12.3 0 22.3 10 22.3 22.3c0 6.2-2.6 12.1-7.1 16.3z"/></svg>
        <InputImage :images="images" ></InputImage>
        <svg class="file-btn" @click="selectFile" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M256 512A256 256 0 1 0 256 0a256 256 0 1 0 0 512zM232 344V280H168c-13.3 0-24-10.7-24-24s10.7-24 24-24h64V168c0-13.3 10.7-24 24-24s24 10.7 24 24v64h64c13.3 0 24 10.7 24 24s-10.7 24-24 24H280v64c0 13.3-10.7 24-24 24s-24-10.7-24-24z"/></svg>
        <input class="file-input" @change="handleFileSelect" ref="inputFile" type="file" accept="image/*" style="display: none;" multiple>
        <input class="input" autocomplete="off" v-model="input" :onpaste="onPaste" @keyup.enter="onSend()" />
        <svg class="input-btn" @click="onSend()" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M498.1 5.6c10.1 7 15.4 19.1 13.5 31.2l-64 416c-1.5 9.7-7.4 18.2-16 23s-18.9 5.4-28 1.6L284 427.7l-68.5 74.1c-8.9 9.7-22.9 12.9-35.2 8.1S160 493.2 160 480V396.4c0-4 1.5-7.8 4.2-10.7L331.8 202.8c5.8-6.3 5.6-16-.4-22s-15.7-6.4-22-.7L106 360.8 17.7 316.6C7.1 311.3 .3 300.7 0 288.9s5.9-22.8 16.1-28.7l448-256c10.7-6.1 23.9-5.5 34 1.4z"/></svg>
    </div>
</template>

<style>

.scroll-btn {
    position: absolute;
    display: flex;
    align-items: center;
    width: 40px;
    height: 40px;
    bottom: 80px;
    right: 24px;
    transition: opacity 0.1s ease-in-out;
}

.messages { 
    position: relative;
    display: flex;
    flex-direction: column;
    flex-grow: 1;
    width: 100%;
    height: 0;
    margin: 0;
    padding: 0;
    overflow: auto;
    background-color: #fff;
}

.form { 
    overflow: hidden; 
    display: flex; 
    justify-content: center;
    align-items: center;
    height: 64px; 
    box-sizing: border-box; 
    background-color: #e7f0f4;
    border-radius: 20px;
}

.file-btn {
    height: 40px;
    width: 40px;
}

path {
    fill: #5e6e82;
}

.input { 
    flex-grow: 1;
    border: none;
    height: 100%;
    background-color: transparent;
    font-size: 24px;
    font-family: 'GyeonggiTitle';
}

.input:focus { 
    outline: none; 
}

.file-btn, .input-btn {
    padding: 0 24px 0 24px;
}

.input-btn { 
    height: 32px;
    width: 32px;
    display: flex; 
    justify-content: center;
    align-items: center;
    margin-right: 4px;
}

path {
    fill: #5e6e82;
}

</style>