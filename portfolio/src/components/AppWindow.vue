<template>
  <!-- 
    The main transition component for the window's appear/disappear animation.
    We use v-show to keep the component in the DOM so we can animate its exit.
  -->
  <Transition name="window-open" appear>
    <div 
      v-show="!isMinimized"
      ref="windowRef"
      class="window" 
      :class="{ 
        maximized: isMaximized, 
        dragging: isDragging,
        resizing: isResizing
      }" 
      :style="windowStyle" 
      @mousedown="focusWindow"
    >
      <!-- 
        Title bar for dragging the window and window controls.
        We use .prevent on mousedown to avoid text selection while dragging.
      -->
      <div class="title-bar" @mousedown.prevent="startDrag">
        <div class="buttons">
          <span class="close" @click="closeWindow"></span>
          <span class="minimize" @click="toggleMinimize"></span>
          <span class="maximize" @click="toggleMaximize"></span>
        </div>
        <span class="title">{{ title }}</span>
      </div>

      <!-- Main content area of the window -->
      <div class="content">
        <slot>
          Default content for {{ title }}.
        </slot>
      </div>

      <!-- Resize handles for each direction -->
      <div 
        v-for="dir in resizeDirections" 
        :key="dir" 
        class="resize-handle" 
        :class="dir"
        @mousedown.prevent="startResize(dir, $event)"
      ></div>
    </div>
  </Transition>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

// Component props
const props = defineProps({
  title: { type: String, default: 'Window' },
  initialX: { type: Number, default: 150 },
  initialY: { type: Number, default: 150 },
  initialWidth: { type: Number, default: 700 },
  initialHeight: { type: Number, default: 500 },
  zIndex: { type: Number, default: 100 }
});

// Component events
const emit = defineEmits(['close', 'focus', 'minimize']);

// Reactive state variables
const x = ref(props.initialX);
const y = ref(props.initialY);
const width = ref(props.initialWidth);
const height = ref(props.initialHeight);
const isMaximized = ref(false);
const isMinimized = ref(false);
const isDragging = ref(false);
const isResizing = ref(false);
const windowRef = ref(null);

// Store pre-maximized state
const preMaximizeState = ref({ x: 0, y: 0, width: 0, height: 0 });

// Directions for resize handles
const resizeDirections = ['top-left', 'top', 'top-right', 'right', 'bottom-right', 'bottom', 'bottom-left', 'left'];

// --- Computed Properties ---

// Dynamically calculates the window's style based on its state
const windowStyle = computed(() => {
  const baseStyle = {
    zIndex: props.zIndex
  };

  // When maximized, the window takes up the full screen.
  // Otherwise, it uses the dynamic x, y, width, and height values.
  if (isMaximized.value) {
    return {
      ...baseStyle,
      top: '0',
      left: '0',
      width: '100vw',
      height: '100vh',
    };
  }
  
  return {
    ...baseStyle,
    top: `${y.value}px`,
    left: `${x.value}px`,
    width: `${width.value}px`,
    height: `${height.value}px`,
  };
});

// --- Window Actions ---

// Emits a 'focus' event to bring the window to the front
function focusWindow() {
  emit('focus');
}

// Toggles the window between maximized and its previous state
function toggleMaximize() {
  focusWindow();
  if (isMaximized.value) {
    // Restore to pre-maximized state
    const { x: prevX, y: prevY, width: prevWidth, height: prevHeight } = preMaximizeState.value;
    x.value = prevX;
    y.value = prevY;
    width.value = prevWidth;
    height.value = prevHeight;
  } else {
    // Save current state and then maximize
    preMaximizeState.value = { x: x.value, y: y.value, width: width.value, height: height.value };
  }
  isMaximized.value = !isMaximized.value;
}

// Toggles the minimized state of the window
function toggleMinimize() {
  isMinimized.value = !isMinimized.value;
  if (isMinimized.value) {
    // Let the parent know this window was minimized
    emit('minimize');
  } else {
    focusWindow();
  }
}

// Closes the window by emitting a 'close' event
function closeWindow() {
  emit('close');
}

// --- Dragging Logic ---

let dragStartX, dragStartY, initialX, initialY;

function startDrag(event) {
  if (isMaximized.value) return;
  focusWindow();
  
  isDragging.value = true;
  dragStartX = event.clientX;
  dragStartY = event.clientY;
  initialX = x.value;
  initialY = y.value;
  
  window.addEventListener('mousemove', onDrag);
  window.addEventListener('mouseup', stopDrag, { once: true });
}

function onDrag(event) {
  if (!isDragging.value) return;
  
  const dx = event.clientX - dragStartX;
  const dy = event.clientY - dragStartY;
  
  // Prevent dragging outside the viewport
  const newX = initialX + dx;
  const newY = initialY + dy;
  const headerHeight = 30; // Assuming a top bar in the OS

  x.value = Math.max(0, Math.min(newX, window.innerWidth - width.value));
  y.value = Math.max(headerHeight, Math.min(newY, window.innerHeight - height.value));
}

function stopDrag() {
  isDragging.value = false;
  window.removeEventListener('mousemove', onDrag);
}


// --- Resizing Logic ---

let resizeStartMouseX, resizeStartMouseY, resizeStartWidth, resizeStartHeight, resizeStartX, resizeStartY;
let currentResizeDir = null;

function startResize(direction, event) {
  if (isMaximized.value) return;
  focusWindow();

  isResizing.value = true;
  currentResizeDir = direction;
  resizeStartMouseX = event.clientX;
  resizeStartMouseY = event.clientY;
  resizeStartWidth = width.value;
  resizeStartHeight = height.value;
  resizeStartX = x.value;
  resizeStartY = y.value;

  window.addEventListener('mousemove', onResize);
  window.addEventListener('mouseup', stopResize, { once: true });
}

function onResize(event) {
  if (!isResizing.value) return;

  const dx = event.clientX - resizeStartMouseX;
  const dy = event.clientY - resizeStartMouseY;
  const minWidth = 350;
  const minHeight = 250;

  let newWidth = resizeStartWidth;
  let newHeight = resizeStartHeight;
  let newX = resizeStartX;
  let newY = resizeStartY;

  if (currentResizeDir.includes('right')) {
    newWidth = Math.max(minWidth, resizeStartWidth + dx);
  }
  if (currentResizeDir.includes('bottom')) {
    newHeight = Math.max(minHeight, resizeStartHeight + dy);
  }
  if (currentResizeDir.includes('left')) {
    const calculatedWidth = resizeStartWidth - dx;
    if (calculatedWidth > minWidth) {
      newWidth = calculatedWidth;
      newX = resizeStartX + dx;
    }
  }
  if (currentResizeDir.includes('top')) {
    const calculatedHeight = resizeStartHeight - dy;
    if (calculatedHeight > minHeight) {
      newHeight = calculatedHeight;
      newY = resizeStartY + dy;
    }
  }
  
  width.value = newWidth;
  height.value = newHeight;
  x.value = newX;
  y.value = newY;
}

function stopResize() {
  isResizing.value = false;
  window.removeEventListener('mousemove', onResize);
}
</script>

<style scoped>
/* Base window styling */
.window {
  position: absolute;
  background: #fdfdfd;
  border-radius: 12px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25), 0 0 15px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  /* Animate position, size, and opacity changes */
  transition: width 0.3s ease, height 0.3s ease, top 0.3s ease, left 0.3s ease, opacity 0.3s ease;
}

/* When dragging or resizing, we want instant feedback, not a transition */
.window.dragging, .window.resizing {
  transition: none;
  user-select: none;
}
.window.dragging .content, .window.resizing .content {
  pointer-events: none;
}

/* Maximized state removes border radius and shadow for a seamless fit */
.window.maximized {
  border-radius: 0;
  border: none;
  box-shadow: none;
}

/* --- Title Bar --- */
.title-bar {
  height: 38px;
  padding: 10px;
  background: linear-gradient(to bottom, #f6f6f6, #e8e8e8);
  border-bottom: 1px solid #ccc;
  display: flex;
  align-items: center;
  /* padding: 0 12px; */
  cursor: move;
  flex-shrink: 0;
}

.title {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: #333;
  text-align: center;
  flex-grow: 1;
  padding-right: 60px; /* Offset for buttons */
}

/* --- Window Buttons (Close, Minimize, Maximize) --- */
.buttons {
  display: flex;
  gap: 8px;
}

.buttons span {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: transform 0.1s ease;
}
.buttons span:hover {
  transform: scale(1.1);
}

.close { background: #ff5f57; border: 1px solid #e24640; }
.minimize { background: #ffbd2e; border: 1px solid #e1a325; }
.maximize { background: #28c940; border: 1px solid #1fa82f; }

/* --- Content Area --- */
.content {
  flex-grow: 1;
  overflow: auto;
  /* padding: 16px; */
  background: #fff;
}

/* --- Resize Handles --- */
.resize-handle {
  position: absolute;
  z-index: 10;
}
.top, .bottom { height: 6px; left: 6px; right: 6px; }
.left, .right { width: 6px; top: 6px; bottom: 6px; }
.top { top: 0; cursor: ns-resize; }
.bottom { bottom: 0; cursor: ns-resize; }
.left { left: 0; cursor: ew-resize; }
.right { right: 0; cursor: ew-resize; }
.top-left { top: 0; left: 0; width: 12px; height: 12px; cursor: nwse-resize; }
.top-right { top: 0; right: 0; width: 12px; height: 12px; cursor: nesw-resize; }
.bottom-left { bottom: 0; left: 0; width: 12px; height: 12px; cursor: nesw-resize; }
.bottom-right { bottom: 0; right: 0; width: 12px; height: 12px; cursor: nwse-resize; }


/* --- Window Open/Close Animation --- */
.window-open-enter-active {
  transition: all 0.25s ease-out;
}
.window-open-leave-active {
  transition: all 0.2s ease-in;
}
.window-open-enter-from,
.window-open-leave-to {
  opacity: 0;
  transform: scale(0.95) translateY(10px);
}
</style>
