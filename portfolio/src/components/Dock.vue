 
  <template>
    <div class="dock">
        <img class="dock-icon" src="../assets/images/dock/finder.png" alt="Finder" @click="toggle('finder')" />
        <img class="dock-icon" src="../assets/images/dock/safari.webp" alt="Safari" @click="toggle('safari')" />
        <img class="dock-icon" src="../assets/images/dock/vs-code.png" alt="Mail" @click="toggle('vs-code')" />
        <img class="dock-icon" src="../assets/images/dock/notes.webp" alt="Notes" @click="toggle('notes')" />
        <img class="dock-icon" src="../assets/images/dock/settings.webp" alt="Settings" @click="toggle('settings')" />
        <img class="dock-icon" src="../assets/images/dock/terminal.webp" alt="Terminal" @click="toggle('terminal')" />
        <img class="dock-icon" src="../assets/images/dock/trash.png" alt="Trash" @click="toggle('trash')" />

    </div>
  </template>
  
  <script setup>
  const emit = defineEmits(['toggle'])
  
  function toggle(appName) {
    emit('toggle', appName)
  }
  </script>
  