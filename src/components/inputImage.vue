<script setup>

const props = defineProps({
    images: [Object]
})

</script>

<template>
    <template v-if="images.length != 0">
        <div class="image-container">
            <template v-for="img, i of props.images" :key="i">
                <img :src="img.url" class="img">
            </template>
        </div>
    </template>
</template>

<style scoped>

.image-container {
    position: absolute;
    bottom: 96px;
    left: 16px;
    background-color: #e7f0f4;
    height: 150px;
    max-width: 80vh;
    padding-right: 16px;
    border-radius: 20px;
    flex-direction: row;
    display: flex;
    align-items: center;
    overflow-x: auto;
    white-space: nowrap;
    scrollbar-width: none;
    -ms-overflow-style: none;
}

.img {
    max-height: 100px;
    padding-left: 16px;
}

</style>