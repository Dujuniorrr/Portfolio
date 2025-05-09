<template>
    <div class="dock">
        <img class="dock-icon" src="../assets/images/finder.png" alt="Finder" @click="toggle('finder')" />
        <img class="dock-icon" src="../assets/images/settings.webp" alt="Settings" @click="toggle('settings')" />
        <img class="dock-icon" src="../assets/images/trash.png" alt="Trash" @click="toggle('trash')" />
    </div>
</template>

<script setup>
const emit = defineEmits(['toggle'])

function toggle(appName) {
    emit('toggle', appName)
}
</script>