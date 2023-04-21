<script setup>

const props = defineProps({
    url: String
})

</script>

<template>
    <img :src="props.url" class="img">
</template>

<style scoped>

.img {
    max-height: 100px;
    padding-left: 16px;
    padding-right: 16px;
}

</style>