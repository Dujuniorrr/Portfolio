<template>
    <div class="menu-bar">
      <div class="menu-left"> Finder File Edit View Go Window Help</div>
      <div class="menu-right">{{ currentTime }}</div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  
  const currentTime = ref('');
  
  onMounted(() => {
    updateClock();
    setInterval(updateClock, 1000);
  });
  
  function updateClock() {
    const now = new Date();
    currentTime.value = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' });
  }
  </script>
  