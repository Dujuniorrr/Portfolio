<template>
  <div class="vs-code-container">
    <!-- Menu Lateral -->
    <div class="sidebar">
      <div class="sidebar-item active">
        <i class="codicon codicon-files"></i>
      </div>
      <div class="sidebar-item">
        <i class="codicon codicon-terminal"></i>
      </div>
      <div class="sidebar-item">
        <i class="codicon codicon-source-control"></i>
      </div>
    </div>

    <!-- Editor de Código -->
    <div class="code-editor">
      <!-- Barra de Abas -->
      <div class="tab-bar">
        <span class="tab" :class="{ active: activeTab === 'index.js' }" @click="setActiveTab('index.js')">
          <Icon icon="simple-icons:javascript" class="icon icon-js" width="10" height="10" /> index.js
        </span>
        <span class="tab" :class="{ active: activeTab === 'App.vue' }" @click="setActiveTab('App.vue')">
          <i class="tab-icon vue-icon"></i> App.vue
        </span>
        <span class="tab" :class="{ active: activeTab === 'README.md' }" @click="setActiveTab('README.md')">
          <i class="tab-icon readme-icon"></i> README.md
        </span>
      </div>
      <!-- Conteúdo do Editor -->
      <div class="editor-content">
        <pre v-if="activeTab === 'index.js'"><code class="language-javascript">
const app = () => {
  console.log('Hello World!');
};
        </code></pre>

        <pre v-if="activeTab === 'App.vue'"><code class="language-html">

        </code></pre>

        <div v-if="activeTab === 'README.md'" class="readme-content">
          <h2>README</h2>
          <p>Este projeto simula um editor de código com um README exibido ao lado.</p>
          <h3>Como usar:</h3>
          <ul>
            <li>Edite os arquivos no painel de código à esquerda.</li>
            <li>O painel à direita exibe um README com informações sobre o projeto.</li>
          </ul>
          <h3>Dependências:</h3>
          <ul>
            <li>Vue 3</li>
            <li>Vue Router</li>
            <li>Vite</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const activeTab = ref('index.js');

const setActiveTab = (tab) => {
  activeTab.value = tab;
};
</script>

<style scoped>
/* Container Principal */
.vs-code-container {
  display: flex;
  height: 100vh;
  /* font-family: 'Courier New', monospace; */
}

/* Sidebar */
.sidebar {
  /* width: 40px; */
  background-color: #333;
  display: flex;
  align-items: start;
  flex-direction: column;
  /* padding-top: 20px; */
}

.icon {
  margin-right: 8px;
  margin-bottom: 4px;
}

.icon-js{
  color: rgb(234, 234, 14);
}

.sidebar-item {
  color: #dcdcdc;
  font-size: 14px;
  padding: 10px 10px;
  cursor: pointer;
  width: 100%;
  text-align: start;
}

.sidebar-item:hover,
.sidebar-item.active {
  background-color: #007acc;
  color: white;
}

.sidebar-icon {
  font-size: 18px;
  display: block;
  margin-bottom: 5px;
}

/* Code Editor */
.code-editor {
  flex-grow: 1;
  background-color: #1e1e1e;
  color: #dcdcdc;
  padding: 0px 2px;
  display: flex;
  flex-direction: column;
}

/* Barra de Abas */
.tab-bar {
  display: flex;
  /* gap: 10px; */
  margin-bottom: 10px;
}

.tab {
  background-color: #333;
  padding: 8px 15px;
  cursor: pointer;
  /* border-radius: 5px; */
  display: flex;
  align-items: center;
  font-size: 0.75rem;
  transition: background-color 0.3s ease;
}

.tab.active {
  background-color: #007acc;
  color: white;
}

.tab:hover {
  background-color: #444;
}

.tab-icon {
  margin-right: 5px;
  font-size: 0.9rem;
}

.js-icon::before {
  content: '\f0c9';
  /* Código para ícone JavaScript */
  font-family: 'Font Awesome 5 Free';
}

.vue-icon::before {
  content: '\f186';
  /* Código para ícone Vue */
  font-family: 'Font Awesome 5 Free';
}

.readme-icon::before {
  content: '\f02d';
  /* Código para ícone README */
  font-family: 'Font Awesome 5 Free';
}

/* Conteúdo do Editor */
.editor-content {
  flex-grow: 1;
  overflow-y: auto;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.editor-content pre {
  margin: 0;
  font-size: 14px;
  line-height: 1.5;
}

/* README */
.readme-content {
  padding: 20px;
  font-size: 0.9rem;
  color: #333;
  background-color: #f9f9f9;
  border-radius: 6px;
}
</style>
