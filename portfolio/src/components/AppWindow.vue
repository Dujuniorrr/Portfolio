<template>
    <div
      class="window"
      :class="{ maximized: isMaximized }"
      :style="windowStyle"
    >
      <div class="title-bar" @mousedown="startDrag">
        <div class="buttons">
          <span class="close" @click="$emit('close')"></span>
          <span class="minimize"></span>
          <span class="maximize" @click="toggleMaximize"></span>
        </div>
        <span class="title">{{ title }}</span>
      </div>
      <div class="content">
        <p>Conteúdo da janela {{ title }}.</p>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed } from 'vue'
  
  const props = defineProps({
    title: String,
    x: { type: Number, default: 100 },
    y: { type: Number, default: 100 }
  })
  
  const isMaximized = ref(false)
  const x = ref(props.x)
  const y = ref(props.y)
  let offsetX = 0, offsetY = 0, dragging = false
  
  const windowStyle = computed(() => {
    return isMaximized.value
      ? { top: 0, left: 0, width: '100vw', height: '100vh' }
      : {
          top: y.value + 'px',
          left: x.value + 'px',
          width: '600px',
          height: '400px'
        }
  })
  
  function toggleMaximize() {
    isMaximized.value = !isMaximized.value
  }
  
  function startDrag(event) {
    if (isMaximized.value) return // não arrasta se estiver maximizada
    dragging = true
    offsetX = event.clientX - x.value
    offsetY = event.clientY - y.value
    document.addEventListener('mousemove', onMouseMove)
    document.addEventListener('mouseup', stopDrag)
  }
  
  function onMouseMove(event) {
    if (dragging) {
      x.value = event.clientX - offsetX
      y.value = event.clientY - offsetY
    }
  }
  
  function stopDrag() {
    dragging = false
    document.removeEventListener('mousemove', onMouseMove)
    document.removeEventListener('mouseup', stopDrag)
  }
  </script>
  
  <style scoped>
  .window {
    position: absolute;
    background: white;
    border: 1px solid #ccc;
    box-shadow: 0 0 10px #0003;
    transition: all 0.2s ease;
    border-radius: 6px;
    overflow: hidden;
  }
  
  .window.maximized {
    border-radius: 0;
  }
  
  .title-bar {
    background: #e0e0e0;
    padding: 6px;
    cursor: move;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .buttons {
    display: flex;
    gap: 6px;
  }
  
  .buttons span {
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
  }
  
  .close {
    background: #ff5f57;
  }
  .minimize {
    background: #ffbd2e;
  }
  .maximize {
    background: #28c840;
  }
  
  .content {
    padding: 16px;
  }
  </style>
  