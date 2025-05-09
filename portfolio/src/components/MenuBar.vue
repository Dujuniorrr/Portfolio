<template>
    <div class="menu-bar">
        <div class="menu-left">
            <div class="menu-item"></div>
            <div class="menu-item active">Finder</div>
            <div class="menu-item">File</div>
            <div class="menu-item">Edit</div>
            <div class="menu-item">View</div>
            <div class="menu-item">Go</div>
            <div class="menu-item">Window</div>
            <div class="menu-item">Help</div>
        </div>
        <div class="menu-right">{{ currentDateTime }}</div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const currentDateTime = ref('');

onMounted(() => {
    updateClock();
    setInterval(updateClock, 1000);
});

function updateClock() {
    const now = new Date();

    const options = {
        weekday: 'short', // Exibe o dia da semana (ex: Mon, Tue)
        // year: 'numeric',
        month: 'short', // Exibe o mês abreviado (ex: Jan, Feb)
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        // second: '2-digit',
        hour12: false, // Formato de 24 horas
    };

    currentDateTime.value = now.toLocaleString('pt-BR', options);
}
</script>

<style>

.active {
    font-weight: bolder;
}

.menu-bar {
  display: flex;
  height: 30px;
  justify-content: space-between;
  align-items: center;
  padding: 4px 8px;
  background: rgba(171, 171, 171, 0.5);
  width: 100%; /* Garantir que a barra de menu ocupe toda a largura */
  box-sizing: border-box; /* Considera o padding na largura total */
}

.menu-left {
  display: flex;
  align-items: center;
  font-family: Arial, sans-serif;
  font-size: 0.9rem; /* Tamanho de fonte reduzido */
  color: white;
}

.menu-item {
  margin-right: 15px; /* Menor espaçamento entre os itens do menu */
  cursor: pointer;
  transition: color 0.3s ease;
}

.menu-item:hover {
  color: #f0f0f0;
}

/* Ícone () em tamanho maior, mas reduzido */
.menu-item:first-child {
  font-size: 1.2rem; /* Reduzido */
  margin-right: 30px; /* Ajuste do espaçamento */
}

.menu-right {
  font-size: 0.8rem; /* Reduzir ainda mais o tamanho da fonte */
  color: white;
  margin-left: 10px;
  white-space: nowrap; /* Impede quebra de linha */
  overflow: hidden; /* Esconde qualquer conteúdo que ultrapasse o limite */
  text-overflow: ellipsis; /* Adiciona "..." se o conteúdo ultrapassar */
}

/* Media Queries para responsividade */
@media (max-width: 768px) {
  .menu-left {
    font-size: 0.9rem; /* Tamanho de fonte menor para telas menores */
  }

  .menu-item {
    margin-right: 10px; /* Menos espaçamento em telas pequenas */
  }

  .menu-item:first-child {
    font-size: 1rem; /* Reduzir ainda mais o ícone em telas pequenas */
    margin-right: 20px; /* Ajuste de espaçamento */
  }

  .menu-bar {
    padding: 8px 10px; /* Menos padding em telas pequenas */
  }
}
</style>
