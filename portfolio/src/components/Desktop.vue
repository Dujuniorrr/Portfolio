<template>
    <div class="desktop">
        <MenuBar />

        <div>

            <AppWindow v-if="windows.finder" title="Finder" :x="positions.finder.x" :y="positions.finder.y"
                :initialWidth="1000"
                :initialHeight="500"
                @close="windows.finder = false" @start-drag="startDrag('finder')" @drag="onDrag"
                @stop-drag="stopDrag" />
            <AppWindow v-if="windows.safari" title="Safari" :x="positions.safari.x" :y="positions.safari.y"
                @close="windows.safari = false" @start-drag="startDrag('safari')" @drag="onDrag"
                @stop-drag="stopDrag" />
            <AppWindow v-if="windows['vs-code']" title="VS Code" :x="positions['vs-code'].x" :y="positions['vs-code'].y"
                :initialWidth="740"
                :initialHeight="500"
                @close="windows['vs-code'] = false" @start-drag="startDrag('vs-code')" @drag="onDrag"
                @stop-drag="stopDrag" />
            <AppWindow v-if="windows.notes" title="Notes" :x="positions.notes.x" :y="positions.notes.y"
                @close="windows.notes = false" @start-drag="startDrag('notes')" @drag="onDrag" @stop-drag="stopDrag" />
            <AppWindow v-if="windows.settings" title="Settings" :x="positions.settings.x" :y="positions.settings.y"
                @close="windows.settings = false" @start-drag="startDrag('settings')" @drag="onDrag"
                @stop-drag="stopDrag" />
            <AppWindow v-if="windows.terminal" title="Terminal" :x="positions.terminal.x" :y="positions.terminal.y"
                @close="windows.terminal = false" @start-drag="startDrag('terminal')" @drag="onDrag"
                @stop-drag="stopDrag" />
            <AppWindow v-if="windows.trash" title="Trash" :x="positions.trash.x" :y="positions.trash.y"
                @close="windows.trash = false" @start-drag="startDrag('trash')" @drag="onDrag" @stop-drag="stopDrag" />

            <Dock @toggle="toggleWindow" />
        </div>
    </div>
</template>

<script setup>
import { reactive } from 'vue'
import MenuBar from './MenuBar.vue'
import Dock from './Dock.vue'
import AppWindow from './AppWindow.vue'

// Estado de visibilidade
const windows = reactive({
    finder: true,
    safari: true,
    'vs-code': true,
    notes: false,
    settings: false,
    terminal: false,
    trash: false,
})

// Estado de posição de cada janela
const positions = reactive({
    finder: { x: 0, y: 80 },
    safari: { x: 400, y: 300 },
    'vs-code': { x: 700, y: 140 },
    notes: { x: 250, y: 160 },
    settings: { x: 300, y: 180 },
    terminal: { x: 350, y: 200 },
    trash: { x: 400, y: 220 },
})

function toggleWindow(name) {
    windows[name] = !windows[name]
}

// Lógica de movimentação
let currentDragging = null

function startDrag(name) {
    currentDragging = name
}

function onDragMove(event) {
    if (!dragging.value) return

    requestAnimationFrame(() => {
        const newX = event.clientX - offsetX
        const newY = event.clientY - offsetY

        const screenWidth = window.innerWidth
        const screenHeight = window.innerHeight
        const menuBarHeight = 30

        const clampedX = Math.min(
            Math.max(0, newX),
            screenWidth - width.value
        )
        const clampedY = Math.min(
            Math.max(menuBarHeight, newY),
            screenHeight - height.value
        )

        x.value = clampedX
        y.value = clampedY
    })
}

function stopDrag() {
    currentDragging = null
}
</script>

<style scoped>
.desktop {
    position: relative;
    height: 100vh;
    width: 100vw;
    overflow: hidden;
    background-color: #f0f0f0;
    background-image: url('../assets//images/background.jpg');

}

.window {
    position: absolute;
    background: white;
    border: 1px solid #ccc;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    border-radius: 6px;
    transition: all 0.2s ease-in-out;
    overflow: hidden;
}
</style>