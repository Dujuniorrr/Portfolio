<template>
  <div class="dock" :class="{ visible: visible }" @mouseenter="emit('force-show')" @mouseleave="emit('force-hide')">
    <div 
      class="dock-item" 
      v-for="(app, index) in apps" 
      :key="index"
      :class="{ active: isActive(app.key) }"
    >
      <div class="tooltip">{{ app.name }}</div>
      <div class="icon-container" @click="toggle(app.key)">
        <img 
          class="dock-icon" 
          :src="app.icon" 
          :alt="app.name"
          :class="{ 'bounce': isActive(app.key) && justActivated[app.key] }"
        />
        <div v-if="isActive(app.key)" class="active-indicator"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { defineProps, ref, watch } from 'vue'

const props = defineProps({
  visible: Boolean,
  activeApps: Array,  // Recebe a lista de apps ativos
})

const emit = defineEmits(['toggle'])

const justActivated = ref({})

const apps = [
  { key: 'finder', name: 'Finder', icon: '/assets/images/dock/finder.png' },
  { key: 'safari', name: 'Safari', icon: '/assets/images/dock/safari.webp' },
  { key: 'vs-code', name: 'VS Code', icon: '/assets/images/dock/vs-code.png' },
  { key: 'notes', name: 'Notes', icon: '/assets/images/dock/notes.webp' },
  { key: 'settings', name: 'Settings', icon: '/assets/images/dock/settings.webp' },
  { key: 'terminal', name: 'Terminal', icon: '/assets/images/dock/terminal.webp' },
  { key: 'trash', name: 'Trash', icon: '/assets/images/dock/trash.png' },
]

// Monitorar mudanças nos apps ativos para animações
watch(() => props.activeApps, (newActiveApps, oldActiveApps) => {
  if (oldActiveApps) {
    // Detectar apps que foram ativados
    newActiveApps.forEach(appKey => {
      if (!oldActiveApps.includes(appKey)) {
        justActivated.value[appKey] = true
        setTimeout(() => {
          justActivated.value[appKey] = false
        }, 600)
      }
    })
  }
}, { immediate: true })

function toggle(appName) {
  console.log(`Dock toggle: ${appName}`)
  emit('toggle', appName)
}

function isActive(key) {
  const active = props.activeApps.includes(key)
  console.log(`Dock isActive: ${key} = ${active}`, { activeApps: props.activeApps })
  return active
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
  background: rgba(255, 255, 255, 0.15);
  border-radius: 20px;
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 1000;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.dock.visible {
  bottom: 20px;
  transform: translateX(-50%) scale(1);
}

.dock-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  transition: all 0.3s ease;
}

.dock-item:hover {
  transform: translateY(-5px);
}

/* .dock-item.active {
  transform: translateY(-8px);
} */

.icon-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
}

.dock-icon {
  width: 65px;
  height: 65px;
  object-fit: contain;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  margin-bottom: 2px;
  filter: drop-shadow(0 2px 8px rgba(0, 0, 0, 0.2));
}

.dock-icon:hover {
  transform: scale(1.15);
  filter: drop-shadow(0 4px 16px rgba(0, 0, 0, 0.3));
}

/* .dock-item.active .dock-icon {
  transform: scale(1.1);
  filter: drop-shadow(0 6/px 20px rgba(0, 0, 0, 0.4));
} */

/* Animação de bounce quando ativado */
.dock-icon.bounce {
  animation: dockBounce 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

@keyframes dockBounce {
  0%, 20%, 50%, 80%, 100% {
    transform: scale(1.1);
  }
  40% {
    transform: scale(1.3);
  }
  60% {
    transform: scale(1.2);
  }
}

.active-indicator {
  position: absolute;
  bottom: -8px;
  width: 6px;
  height: 6px;
  background: linear-gradient(135deg, #00ff88, #00ccff);
  border-radius: 50%;
  box-shadow: 0 0 10px rgba(0, 255, 136, 0.5);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.7;
    transform: scale(1.2);
  }
}

.tooltip {
  position: absolute;
  bottom: 80px;
  background: linear-gradient(135deg, #333, #555);
  color: white;
  padding: 8px 12px;
  font-size: 12px;
  border-radius: 8px;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: all 0.3s ease;
  z-index: 10;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  transform: translateY(10px);
}

.dock-item:hover .tooltip {
  opacity: 1;
  transform: translateY(0);
}

/* Melhorar responsividade */
@media (max-width: 768px) {
  .dock {
    gap: 12px;
    padding: 6px 12px;
  }
  
  .dock-icon {
    width: 50px;
    height: 50px;
  }
  
  .tooltip {
    font-size: 11px;
    padding: 6px 10px;
  }
}
</style>
