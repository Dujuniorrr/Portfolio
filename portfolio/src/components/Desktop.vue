<template>
    <div class="desktop">
      <MenuBar />
      <AppWindow v-if="windows.finder" title="Finder" @close="windows.finder = false" />
      <AppWindow v-if="windows.settings" title="Settings" :x="200" :y="150" @close="windows.settings = false" />
      <AppWindow v-if="windows.trash" title="Trash" :x="300" :y="180" @close="windows.trash = false" />
      <Dock @toggle="toggleWindow" />
    </div>
  </template>
  
  <script setup>
  import { reactive } from 'vue'
  import MenuBar from './MenuBar.vue'
  import Dock from './Dock.vue'
  import AppWindow from './AppWindow.vue'
  
  const windows = reactive({
    finder: false,
    settings: false,
    trash: false
  })
  
  function toggleWindow(name) {
    windows[name] = !windows[name]
  }
  </script>
  