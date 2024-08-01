<template>
    <div class="w-full text-center">
        <p class="text-lg font-bold">{{ timestamp }}</p>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const timestamp = ref(new Date().toLocaleTimeString());

onMounted(() => {
    setInterval(() => {
        timestamp.value = new Date().toLocaleTimeString();
    }, 1000);
});
</script>