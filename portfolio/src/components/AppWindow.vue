<template>
  <div class="window" :class="{ maximized: isMaximized, minimized: isMinimized }" :style="windowStyle" ref="windowRef">
    <div class="title-bar" @mousedown.stop.prevent="startDrag">
      <div class="buttons">
        <span class="close" @click="$emit('close')"></span>
        <span class="minimize" @click="toggleMinimize"></span>
        <span class="maximize" @click="toggleMaximize"></span>
      </div>
      <span class="title">{{ title }}</span>
    </div>


    <div class="content" v-show="!isMinimized">
      <slot>Conteúdo da janela {{ title }}.</slot>
    </div>

    <!-- Resizers -->
    <div v-for="dir in resizeDirs" :key="dir" class="resize-handle" :class="dir"
      @mousedown.stop.prevent="startResize(dir, $event)"></div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// Props now include initial position and size
const props = defineProps({
  title: { type: String, default: '' },
  x: { type: Number, default: 100 },
  y: { type: Number, default: 100 },
  initialWidth: { type: Number, default: 600 },
  initialHeight: { type: Number, default: 400 }
})
const emit = defineEmits(['close', 'maximized', 'minimized'])

const isMaximized = ref(false)
const isMinimized = ref(false)
const x = ref(props.x)
const y = ref(props.y)
const width = ref(props.initialWidth)
const height = ref(props.initialHeight)
const dragging = ref(false)
const resizing = ref(false)
const dir = ref(null)

let startMouseX, startMouseY, startX, startY, startWidth, startHeight
const resizeDirs = ['top-left', 'top', 'top-right', 'right', 'bottom-right', 'bottom', 'bottom-left', 'left']

const windowStyle = computed(() => {
  if (isMaximized.value) {
    return { top: '30px', left: '0', width: '100vw', height: 'calc(100vh - 30px)' }
  }
  return {
    top: `${y.value}px`,
    left: `${x.value}px`,
    width: `${width.value}px`,
    height: `${height.value}px`
  }
})

function toggleMaximize() {
  if (isMaximized.value) {
    x.value = startX
    y.value = startY
    width.value = startWidth
    height.value = startHeight
  } else {
    startX = x.value
    startY = y.value
    startWidth = width.value
    startHeight = height.value
    x.value = 0
    y.value = 0
    width.value = window.innerWidth
    height.value = window.innerHeight - 30
  }
  isMaximized.value = !isMaximized.value
}

function toggleMinimize() {
  if (!isMinimized.value) {
    // Salvar estado antes de minimizar
    startX = x.value
    startY = y.value
    startWidth = width.value
    startHeight = height.value
    isMinimized.value = true
    width.value = 200
    height.value = 30
    y.value = window.innerHeight - 30
    emit('maximized');
  } else {
    x.value = startX
    y.value = startY
    width.value = startWidth
    height.value = startHeight
    isMinimized.value = false
    emit('minimized')
  }
}

function startDrag(event) {
  if (isMaximized.value || isMinimized.value) return
  dragging.value = true
  startMouseX = event.clientX
  startMouseY = event.clientY
  startX = x.value
  startY = y.value
  window.addEventListener('mousemove', onDrag)
  window.addEventListener('mouseup', stopDrag)
}

function onDrag(e) {
  if (!dragging.value) return
  x.value = Math.max(0, Math.min(startX + (e.clientX - startMouseX), window.innerWidth - width.value))
  y.value = Math.max(30, Math.min(startY + (e.clientY - startMouseY), window.innerHeight - height.value))
}

function stopDrag() {
  dragging.value = false
  window.removeEventListener('mousemove', onDrag)
  window.removeEventListener('mouseup', stopDrag)
}

function startResize(direction, event) {
  if (isMaximized.value) return
  resizing.value = true
  dir.value = direction
  startMouseX = event.clientX
  startMouseY = event.clientY
  startX = x.value
  startY = y.value
  startWidth = width.value
  startHeight = height.value
  window.addEventListener('mousemove', onResize)
  window.addEventListener('mouseup', stopResize)
}

function onResize(e) {
  if (!resizing.value) return
  const dx = e.clientX - startMouseX
  const dy = e.clientY - startMouseY
  const minW = 200, minH = 150
  let nx = startX, ny = startY, nw = startWidth, nh = startHeight

  if (dir.value.includes('right')) nw = Math.max(minW, startWidth + dx)
  if (dir.value.includes('bottom')) nh = Math.max(minH, startHeight + dy)
  if (dir.value.includes('left')) {
    nw = Math.max(minW, startWidth - dx)
    nx = startX + (startWidth - nw)
  }
  if (dir.value.includes('top')) {
    nh = Math.max(minH, startHeight - dy)
    ny = startY + (startHeight - nh)
  }

  x.value = Math.max(0, nx)
  y.value = Math.max(30, ny)
  width.value = Math.min(window.innerWidth - x.value, nw)
  height.value = Math.min(window.innerHeight - y.value, nh)
}

function stopResize() {
  resizing.value = false
  window.removeEventListener('mousemove', onResize)
  window.removeEventListener('mouseup', stopResize)
}
</script>

<style scoped>
.window {
  position: absolute;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  border-radius: 6px;
  overflow: hidden;
}

.title-bar {
  height: 30px;
  background: #2e2e2e;
  color: white;
  display: flex;
  align-items: center;
  padding: 0 10px;
  cursor: move;
}

.title {
  font-family: Arial, sans-serif;
  font-size: 0.8rem;
  /* Tamanho de fonte reduzido */
  font-weight: 400;
}

.buttons span {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 6px;
  display: inline-block;
  cursor: pointer;
}

.close {
  background: #e74c3c;
}

.minimize {
  background: #f39c12;
}

.maximize {
  background: #2ecc71;
}

.content {
  flex: 1;
  overflow: auto;
}

.resize-handle {
  position: absolute;
  background: transparent;
  z-index: 5;
}

.top-left {
  top: 0;
  left: 0;
  width: 10px;
  height: 10px;
  cursor: nwse-resize;
}

.top {
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 10px;
  height: 5px;
  cursor: ns-resize;
}

.top-right {
  top: 0;
  right: 0;
  width: 10px;
  height: 10px;
  cursor: nesw-resize;
}

.right {
  top: 50%;
  right: 0;
  transform: translateY(-50%);
  width: 5px;
  height: 10px;
  cursor: ew-resize;
}

.bottom-right {
  bottom: 0;
  right: 0;
  width: 10px;
  height: 10px;
  cursor: nwse-resize;
}

.bottom {
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 10px;
  height: 5px;
  cursor: ns-resize;
}

.bottom-left {
  bottom: 0;
  left: 0;
  width: 10px;
  height: 10px;
  cursor: nesw-resize;
}

.left {
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  width: 5px;
  height: 10px;
  cursor: ew-resize;
}

.window.maximized {
  border-radius: 0;
}

.window.minimized {
  display: none;
}
</style>