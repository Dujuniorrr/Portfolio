<template>
  <div class="desktop">
    <MenuBar />
    <div class="windows-container">
      <!-- Finder -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows.finder" 
          title="Finder" 
          :x="positions.finder.x" 
          :y="positions.finder.y"
          :initialWidth="1000" 
          :initialHeight="500" 
          :zIndex="getZIndex('finder')"
          @close="closeWindow('finder')"
          @start-drag="startDrag('finder')" 
          @focus="focusWindow('finder')"
          @minimize-toggle="handleMinimizeToggle('finder', $event)"
        >
        <FinderContent />
        </AppWindow>
      </Transition>

      <!-- Safari -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows.safari" 
          title="Safari" 
          :x="positions.safari.x" 
          :y="positions.safari.y"
          :zIndex="getZIndex('safari')"
          @close="closeWindow('safari')" 
          @start-drag="startDrag('safari')" 
          @focus="focusWindow('safari')"
          @minimize-toggle="handleMinimizeToggle('safari', $event)"
        />
      </Transition>

      <!-- VS Code -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows['vs-code']" 
          title="VS Code" 
          :x="positions['vs-code'].x" 
          :y="positions['vs-code'].y"
          :initialWidth="740" 
          :initialHeight="500" 
          :zIndex="getZIndex('vs-code')"
          @close="closeWindow('vs-code')"
          @start-drag="startDrag('vs-code')"
          @focus="focusWindow('vs-code')"
          @minimize-toggle="handleMinimizeToggle('vs-code', $event)"
        >
          <VsCodeContent />
        </AppWindow>
      </Transition>

      <!-- Notes -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows.notes" 
          title="Notes" 
          :x="positions.notes.x" 
          :y="positions.notes.y"
          :zIndex="getZIndex('notes')"
          @close="closeWindow('notes')" 
          @start-drag="startDrag('notes')" 
          @drag="onDrag" 
          @stop-drag="stopDrag" 
          @focus="focusWindow('notes')"
          @minimize-toggle="handleMinimizeToggle('notes', $event)"
        />
      </Transition>

      <!-- Settings -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows.settings" 
          title="Settings" 
          :x="positions.settings.x" 
          :y="positions.settings.y"
          :zIndex="getZIndex('settings')"
          @close="closeWindow('settings')" 
          @start-drag="startDrag('settings')" 
          @focus="focusWindow('settings')"
          @minimize-toggle="handleMinimizeToggle('settings', $event)"
        />
      </Transition>

      <!-- Terminal -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows.terminal" 
          title="Terminal" 
          :x="positions.terminal.x" 
          :y="positions.terminal.y"
          :zIndex="getZIndex('terminal')"
          @close="closeWindow('terminal')" 
          @start-drag="startDrag('terminal')"
          @focus="focusWindow('terminal')"
          @minimize-toggle="handleMinimizeToggle('terminal', $event)"
        >
          <TerminalContent />
        </AppWindow>
      </Transition>

      <!-- Trash -->
      <Transition name="window" appear>
        <AppWindow 
          v-if="windows.trash" 
          title="Trash" 
          :x="positions.trash.x" 
          :y="positions.trash.y"
          :zIndex="getZIndex('trash')"
          @close="closeWindow('trash')" 
          @start-drag="startDrag('trash')" 
          @drag="onDrag" 
          @stop-drag="stopDrag" 
          @focus="focusWindow('trash')"
          @minimize-toggle="handleMinimizeToggle('trash', $event)"
        />
      </Transition>
    </div>

    <Dock @toggle="toggleWindow" :visible="shouldShowDock" :activeApps="activeApps" />
  </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted, nextTick } from 'vue'
import MenuBar from './MenuBar.vue'
import Dock from './Dock.vue'
import AppWindow from './AppWindow.vue'
import VsCodeContent from './contents/VsCodeContent.vue'
import TerminalContent from './contents/TerminalContent.vue'
import FinderContent from './contents/FinderContent.vue'

const forceVisible = ref(true)
const isMouseNearBottom = ref(false)

const windows = reactive({
  finder: true,
  safari: false,
  'vs-code': false,
  notes: false,
  settings: false,
  terminal: false,
  trash: false,
})

// Estado das janelas minimizadas
const minimizedWindows = reactive({
  finder: false,
  safari: false,
  'vs-code': false,
  notes: false,
  settings: false,
  terminal: false,
  trash: false,
})

// Sistema de z-index dinâmico
const zIndexCounter = ref(1000)
const windowZIndexes = reactive({
  finder: 1000,
  safari: 1001,
  'vs-code': 1002,
  notes: 1003,
  settings: 1004,
  terminal: 1005,
  trash: 1006,
})

const shouldShowDock = computed(() => {
  const hasOpenWindow = Object.values(windows).some(v => v)
  const hasVisibleWindow = Object.values(windows).some((v, i) => v && !Object.values(minimizedWindows)[i])
  const result = forceVisible.value || isMouseNearBottom.value || !hasOpenWindow || !hasVisibleWindow
  console.log('shouldShowDock:', result, { forceVisible: forceVisible.value, isMouseNearBottom: isMouseNearBottom.value, hasOpenWindow, hasVisibleWindow })
  return result
})

function handleMouseMove(event) {
  isMouseNearBottom.value = event.clientY >= window.innerHeight - 100
}

onMounted(() => {
  forceVisible.value = true
  setTimeout(() => {
    forceVisible.value = false
    window.addEventListener('mousemove', handleMouseMove)
  }, 1000)
})

async function toggleWindow(name) {
  console.log(`toggleWindow: ${name}`, {
    isOpen: windows[name],
    isMinimized: minimizedWindows[name]
  })
  
  if (windows[name]) {
    if (minimizedWindows[name]) {
      // Se a janela está minimizada, restaurar
      console.log(`Restaurando janela minimizada: ${name}`)
      minimizedWindows[name] = false
      // A janela já está aberta, só precisa ser restaurada
      focusWindow(name)
    } else {
      // Se a janela está aberta e visível, fechar
      console.log(`Fechando janela visível: ${name}`)
      await closeWindow(name)
    }
  } else {
    // Se a janela está fechada, abrir com animação
    console.log(`Abrindo nova janela: ${name}`)
    windows[name] = true
    minimizedWindows[name] = false
    await nextTick()
    
    // Posicionar a janela em uma posição visível
    ensureWindowVisibility(name)
    
    // Focar a janela recém-aberta
    focusWindow(name)
  }
}

async function closeWindow(name) {
  // Adicionar delay para permitir animação de fechamento
  await new Promise(resolve => setTimeout(resolve, 200))
  windows[name] = false
  minimizedWindows[name] = false // Resetar estado de minimização
  
  // Resetar z-index da janela fechada
  resetZIndex(name)
}

function ensureWindowVisibility(name) {
  // Verificar se a janela está visível na tela
  const pos = positions[name]
  const windowWidth = 600
  const windowHeight = 400
  const menuHeight = 30 // Altura do menu superior
  
  // Se a janela estiver fora da tela, reposicionar
  if (pos.x + windowWidth > window.innerWidth || pos.x < 0) {
    pos.x = Math.max(0, Math.min(window.innerWidth - windowWidth, 100))
  }
  
  if (pos.y + windowHeight > window.innerHeight || pos.y < menuHeight) {
    pos.y = Math.max(menuHeight, Math.min(window.innerHeight - windowHeight, 100))
  }
}

// Função para focar uma janela (elevar para o topo)
function focusWindow(name) {
  if (windows[name]) {
    // Incrementar o contador global
    zIndexCounter.value++
    
    // Atualizar o z-index da janela específica
    windowZIndexes[name] = zIndexCounter.value
  }
}

// Função para obter o z-index de uma janela
function getZIndex(name) {
  return windowZIndexes[name] || 1000
}

// Função para resetar z-index quando janela é fechada
function resetZIndex(name) {
  const baseIndex = 1000
  const windowNames = Object.keys(windows)
  const index = windowNames.indexOf(name)
  if (index !== -1) {
    windowZIndexes[name] = baseIndex + index
  }
  // Resetar também o estado de minimização
  minimizedWindows[name] = false
}

// Função para debug da minimização
function handleMinimizeToggle(name, minimized) {
  console.log(`handleMinimizeToggle: ${name}`, { minimized })
  minimizedWindows[name] = minimized
  console.log(`Estado atualizado:`, minimizedWindows)
  
  // Forçar re-render se necessário
  if (minimized) {
    console.log(`Janela ${name} minimizada`)
  } else {
    console.log(`Janela ${name} restaurada`)
  }
}

const activeApps = computed(() => {
  return Object.keys(windows).filter(app => windows[app])
})

const visibleApps = computed(() => {
  const result = Object.keys(windows).filter(app => windows[app] && !minimizedWindows[app])
  console.log('visibleApps:', result, { windows, minimizedWindows })
  return result
})

// Estado de posição de cada janela com posições mais organizadas
const positions = reactive({
  finder: { x: 50, y: 80 },
  safari: { x: 200, y: 130 },
  'vs-code': { x: 350, y: 80 },
  notes: { x: 500, y: 180 },
  settings: { x: 150, y: 280 },
  terminal: { x: 400, y: 330 },
  trash: { x: 600, y: 230 },
})

// Funções de drag (mantidas para compatibilidade)
function startDrag(name) {
  // Focar a janela quando começar a arrastar
  focusWindow(name)
}

function onDrag() {
  // Implementação do drag se necessário
}

function stopDrag() {
  // Implementação do drag se necessário
}
</script>

<style scoped>
.desktop {
  position: relative;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  background-color: #f0f0f0;
  background-image: url('/assets//images/background.jpg');
  background-size: cover;
  background-position: center;
}

.windows-container {
  position: relative;
  width: 100%;
  height: 100%;
  margin: 30px 0px;
}

/* Animações para as janelas - como no macOS */
.window-enter-active {
  transition: all 0.3s ease-out;
}

.window-leave-active {
  transition: all 0.2s ease-in;
}

.window-enter-from {
  opacity: 0;
  transform: scale(0.9) translateY(20px);
}

.window-leave-to {
  opacity: 0;
  transform: scale(0.9) translateY(-20px);
}

/* Melhorar a aparência geral */
.desktop::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.1);
  pointer-events: none;
}
</style>