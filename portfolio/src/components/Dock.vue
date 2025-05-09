<template>
  <div class="dock" :class="{ visible: visible }" @mouseenter="emit('force-show')" @mouseleave="emit('force-hide')">
    <div class="dock-item" v-for="(app, index) in apps" :key="index">
      <div class="tooltip">{{ app.name }}</div>
      <img class="dock-icon" :src="app.icon" :alt="app.name" @click="toggle(app.key)" />
      <div v-if="isActive(app.key)" class="active-indicator"></div>
    </div>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'

const props  = defineProps({
  visible: Boolean,
  activeApps: Array,  // Recebe a lista de apps ativos
})

const emit = defineEmits(['toggle'])

const apps = [

  { key: 'finder', name: 'Finder', icon: '/assets/images/dock/finder.png' },
  { key: 'safari', name: 'Safari', icon: '/assets/images/dock/safari.webp' },
  { key: 'vs-code', name: 'VS Code', icon: '/assets/images/dock/vs-code.png' },
  { key: 'notes', name: 'Notes', icon: '/assets/images/dock/notes.webp' },
  { key: 'settings', name: 'Settings', icon: '/assets/images/dock/settings.webp' },
  { key: 'terminal', name: 'Terminal', icon: '/assets/images/dock/terminal.webp' },
  { key: 'trash', name: 'Trash', icon: '/assets/images/dock/trash.png' },

]

function toggle(appName) {
  emit('toggle', appName)
}

function isActive(key) {
  return props.activeApps.includes(key)  // Verifica se o app está ativo
}
</script>


<style scoped>
.dock {
  position: fixed;
  bottom: -100px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 16px;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  backdrop-filter: blur(10px);
  box-shadow: 0 0 10px #0005;
  transition: bottom 0.3s ease;
  z-index: 1000;
}

.dock.visible {
  bottom: 10px;
}

.dock-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.app-name {
  font-size: 12px;
  color: #fff;
  margin-bottom: 5px;
}

.dock-icon {
  width: 65px;
  height: 65px;
  object-fit: contain;
  transition: transform 0.2s ease;
  cursor: pointer;
  margin-bottom: 2px;

}

.dock-icon:hover {
  transform: scale(1.2);
}

.active-indicator {
  position: absolute;
  bottom: -5px;
  width: 4px;
  height: 4px;
  margin-top: 2px;
  background-color: white;
  border-radius: 50%;
}

.tooltip {
  position: absolute;
  bottom: 80px;
  background-color: #333333b6;
  color: white;
  padding: 4px 8px;
  font-size: 12px;
  border-radius: 4px;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s ease;
  z-index: 10;
}

.dock-item:hover .tooltip {
  opacity: 1;
}

</style>
