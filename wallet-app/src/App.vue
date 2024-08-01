<template>
  <div id="app-container"
    class="container mx-auto p-4 flex flex-col justify-center items-center h-screen space-y-4 opacity-0 transition-opacity duration-1000 ease-in-out">
    <div class="transform scale-50 transition-transform duration-300 ease-in-out">
      <LiveTimestamp />
    </div>
    <div class="transform scale-50 transition-transform duration-250 ease-in-out">
      <Wallet />
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted } from 'vue';
import LiveTimestamp from './components/LiveTimestamp.vue';
import Wallet from './components/Wallet.vue';

onMounted(() => {
  setTimeout(() => {
    const container = document.getElementById('app-container');
    container.classList.remove('opacity-0');
    container.classList.add('opacity-100');

    const components = container.querySelectorAll('.transform');
    components.forEach(component => {
      component.classList.remove('scale-50');
      component.classList.add('scale-100');
    });
  }, 100);
});

</script>
