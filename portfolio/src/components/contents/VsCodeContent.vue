<template>
  <div class="vs-code-container">
    <!-- Sidebar -->
    <div class="sidebar">
      <!-- Activity Bar -->
      <div class="activity-bar">
        <div class="activity-item active" title="Explorer">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"></path><polyline points="14 2 14 8 20 8"></polyline></svg>
        </div>
        <div class="activity-item" title="Search">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
        </div>
        <div class="activity-item" title="Source Control">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="18" cy="18" r="3"></circle><circle cx="6" cy="6" r="3"></circle><path d="M13 6h3a2 2 0 0 1 2 2v7"></path><line x1="6" y1="9" x2="6" y2="21"></line></svg>
        </div>
        <div class="activity-item" title="Run and Debug">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
        </div>
        <div class="activity-item" title="Extensions">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3zM7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"></path></svg>
        </div>
      </div>

      <!-- Explorer Panel -->
      <div class="explorer-panel">
         <!-- Open Editors Section -->
        <div class="panel-section">
            <div class="panel-header" @click="toggleSection('open')">
                 <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" class="chevron" :class="{ 'rotated': visibleSections.open }">
                    <path d="M7 10l5 5 5-5z"/>
                </svg>
                <span>EDITORES ABERTOS</span>
            </div>
             <div class="file-list" v-show="visibleSections.open">
                <div v-for="tab in openTabs" :key="tab.id" class="file" :class="{ active: activeTabId === tab.id }" @click="setActiveTab(tab.id)">
                    <component :is="getIcon(tab.type)" class="file-icon" />
                    <span>{{ tab.id }}</span>
                </div>
            </div>
        </div>
        
        <!-- File Tree Section -->
        <div class="panel-section">
            <div class="panel-header" @click="toggleSection('files')">
                <div class="panel-title">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" class="chevron" :class="{ 'rotated': visibleSections.files }">
                        <path d="M7 10l5 5 5-5z"/>
                    </svg>
                    <span>PORTFOLIO</span>
                </div>
                <div class="panel-actions">
                    <button title="Novo Arquivo"><svg width="16" height="16" viewBox="0 0 24 24"><path fill="currentColor" d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8l-6-6zm-1 9h-2v2H9v-2H7v-2h2V9h2v2h2v2z"/></svg></button>
                    <button title="Nova Pasta"><svg width="16" height="16" viewBox="0 0 24 24"><path fill="currentColor" d="M10 4H4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V8a2 2 0 0 0-2-2h-8l-2-2z"/></svg></button>
                    <button title="Atualizar"><svg width="16" height="16" viewBox="0 0 24 24"><path fill="currentColor" d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></button>
                    <button title="Recolher Pastas"><svg width="16" height="16" viewBox="0 0 24 24"><path fill="currentColor" d="M19.5 3.09L15 7.59V4h-2v7h7v-2h-3.59l4.5-4.5L19.5 3.09zM4 19h5v-2H5.41l4.5-4.5l-1.41-1.41l-4.5 4.5V10H2v7h7v-2H4z"/></svg></button>
                </div>
            </div>
            <div class="file-tree" v-show="visibleSections.files">
              <div v-for="item in fileTree" :key="item.name" class="tree-item">
                  <div v-if="!item.children" class="file" @click="openFile(item.name, item.type)">
                      <component :is="getIcon(item.type)" class="file-icon" />
                      <span>{{ item.name }}</span>
                  </div>
                  <div v-else class="folder">
                      <div class="folder-header" @click="toggleFolder(item.name)">
                           <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" class="chevron" :class="{ 'rotated': openFolders[item.name] }">
                              <path d="M7 10l5 5 5-5z"/>
                          </svg>
                          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" class="folder-icon"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"></path></svg>
                          <span>{{ item.name }}</span>
                      </div>
                      <div class="folder-content" v-show="openFolders[item.name]">
                          <div v-for="child in item.children" :key="child.name" class="tree-item">
                              <div v-if="!child.children" class="file" @click="openFile(child.name, child.type)">
                                <component :is="getIcon(child.type)" class="file-icon" />
                                <span>{{ child.name }}</span>
                              </div>
                              <div v-else class="folder">
                                 <div class="folder-header" @click="toggleFolder(child.name)">
                                     <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" class="chevron" :class="{ 'rotated': openFolders[child.name] }">
                                        <path d="M7 10l5 5 5-5z"/>
                                    </svg>
                                    <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" class="folder-icon"><path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"></path></svg>
                                    <span>{{ child.name }}</span>
                                 </div>
                                 <div class="folder-content" v-show="openFolders[child.name]">
                                     <div v-for="grandchild in child.children" :key="grandchild.name" class="file" @click="openFile(grandchild.name, grandchild.type)">
                                        <component :is="getIcon(grandchild.type)" class="file-icon" />
                                        <span>{{ grandchild.name }}</span>
                                     </div>
                                 </div>
                              </div>
                          </div>
                      </div>
                  </div>
              </div>
            </div>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
      <!-- Tab Bar -->
      <div class="tab-bar">
        <div v-for="(tab, index) in openTabs" :key="tab.id" class="tab" :class="{ active: activeTabId === tab.id }" @click="setActiveTab(tab.id)">
          <component :is="getIcon(tab.type)" class="file-icon" />
          <span>{{ tab.id }}</span>
          <button class="tab-close" @click.stop="closeTab(index)">×</button>
        </div>
      </div>

      <!-- Editor Content -->
      <div class="editor-content">
        <div class="line-numbers">
          <div v-for="n in 50" :key="n" class="line-number">{{ n }}</div>
        </div>
        <div class="code-content">
          <div v-if="activeTabContent" class="code-file" v-html="activeTabContent.content"></div>
          <div v-else class="welcome-message">
              <p>Selecione um arquivo para começar.</p>
              <p>Abra um arquivo do explorador à esquerda.</p>
          </div>
        </div>
      </div>
      
      <!-- Status Bar -->
      <div class="status-bar">
          <div class="status-left">
              <div class="status-item">
                  <svg width="12" height="12" viewBox="0 0 24 24"><path fill="currentColor" d="M2.5 10.5A.5.5 0 0 1 3 10h1a.5.5 0 0 1 .5.5v4a.5.5 0 0 1-1 0v-3.5h-.5a.5.5 0 0 1-.5-.5zm5 0a.5.5 0 0 1 .5-.5h1a.5.5 0 0 1 .5.5v4a.5.5 0 0 1-1 0v-3.5h-.5a.5.5 0 0 1-.5-.5zm4.5.5a.5.5 0 0 0 0-1h-1a.5.5 0 0 0 0 1h1zm2 0a.5.5 0 0 0 0-1h-1a.5.5 0 0 0 0 1h1zm2 0a.5.5 0 0 0 0-1h-1a.5.5 0 0 0 0 1h1zm2 0a.5.5 0 0 0 0-1h-1a.5.5 0 0 0 0 1h1z"/></svg>
                  <span>main</span>
              </div>
              <div class="status-item">
                  <svg width="12" height="12" viewBox="0 0 16 16"><path fill="currentColor" d="M8 9.5a1.5 1.5 0 1 0 0-3a1.5 1.5 0 0 0 0 3z"/><path fill-rule="evenodd" d="M8 0a8 8 0 1 0 0 16A8 8 0 0 0 8 0zM1.5 8a6.5 6.5 0 1 1 13 0a6.5 6.5 0 0 1-13 0z"/></svg>
                  <span>0</span>
                  <svg width="12" height="12" viewBox="0 0 16 16"><path fill="currentColor" d="M7.354 3.646a.5.5 0 0 1 0 .708L4.707 7H12.5a.5.5 0 0 1 0 1H4.707l2.647 2.646a.5.5 0 0 1-.708.708l-3.5-3.5a.5.5 0 0 1 0-.708l3.5-3.5a.5.5 0 0 1 .708 0z"/></svg>
                  <span>0</span>
              </div>
          </div>
          <div class="status-right">
              <div class="status-item">Ln 1, Col 1</div>
              <div class="status-item">Espaços: 2</div>
              <div class="status-item">UTF-8</div>
              <div class="status-item">{ } {{ activeTabContent?.type.toUpperCase() || '' }}</div>
              <div class="status-item">
                  <svg width="14" height="14" viewBox="0 0 16 16"><path fill="currentColor" d="M8 9.967A2 2 0 0 0 9.732 9H11a3 3 0 0 1-2.268 2.882l.884 1.237a.5.5 0 0 1-.832.562L8 12.426l-.783 1.256a.5.5 0 0 1-.832-.562l.884-1.237A3 3 0 0 1 5 9h1.268A2 2 0 0 0 8 9.967zM4.5 2A1.5 1.5 0 0 0 3 3.5v5A1.5 1.5 0 0 0 4.5 10h7A1.5 1.5 0 0 0 13 8.5v-5A1.5 1.5 0 0 0 11.5 2h-7z"/></svg>
              </div>
          </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, markRaw } from 'vue';

// --- ICONS ---
const IconVue = { template: `<svg width="16" height="16" viewBox="0 0 256 221.7" fill="#42b883"><path d="M204.8 0H256L128 220.8 0 0h97.9L128 51.2 157.4 0h47.4z"/><path d="M0 0l128 220.8L256 0h-51.2L128 132.5 51.2 0H0z" fill="#35495e"/><path d="M51.2 0L128 133.1 204.8 0h-47.4L128 51.2 97.9 0H51.2z" fill="#42b883"/></svg>` };
const IconJS = { template: `<svg width="16" height="16" viewBox="0 0 128 128" fill="#f7df1e"><path d="M1.28 1.28h125.44v125.44H1.28z"/><path d="M115.33 95.11c-1.33 7.33-6.22 12.89-14.89 12.89-8.67 0-13.78-4.89-16.44-12.44l11.11-6.67c1.11 3.11 2.89 5.33 6 5.33 2.89 0 4.44-1.56 4.44-4.22 0-2.22-.89-3.56-4.67-5.11l-2.67-1.11c-6.22-2.67-10.44-7.11-10.44-14.22 0-7.11 5.33-12.44 13.33-12.44 6.22 0 10.67 2.89 13.11 9.56l-10.67 6.44c-.67-2.22-2-3.56-4.22-3.56-2 0-3.33 1.11-3.33 3.33 0 2.22.89 3.11 4 4.67l2.67 1.11c7.11 2.89 11.33 6.89 11.33 14.44zm-54.89-35.11c0-8.22 6.22-13.56 14.89-13.56 8.67 0 14.67 5.33 14.67 13.56 0 8.22-6 13.56-14.67 13.56-8.67 0-14.89-5.33-14.89-13.56zm11.33 0c0-2.89 1.78-5.11 3.56-5.11 2 0 3.56 2.22 3.56 5.11 0 2.89-1.56 5.11-3.56 5.11-1.78 0-3.56-2.22-3.56-5.11z"/></svg>` };
const IconJSON = { template: `<svg width="16" height="16" viewBox="0 0 24 24" fill="#f7df1e"><path d="M6,15V9H8.5C9.88,9 11,10.12 11,11.5V12.5C11,13.88 9.88,15 8.5,15H6M8,13.5H8.5C9.05,13.5 9.5,13.05 9.5,12.5V11.5C9.5,10.95 9.05,10.5 8.5,10.5H8V13.5M13,9H18V10.5H14.5V12H17V13.5H14.5V15H13V9M20,9V15H21.5V10.5H23V9H20Z" fill="#e6e6e6"/></svg>` };
const IconMD = { template: `<svg width="16" height="16" viewBox="0 0 24 24"><path d="M20.5,8 L20.5,16 L3.5,16 L3.5,8 L20.5,8 Z M20.5,6 L3.5,6 C2.395,6 1.5,6.895 1.5,8 L1.5,16 C1.5,17.105 2.395,18 3.5,18 L20.5,18 C21.605,18 22.5,17.105 22.5,16 L22.5,8 C22.5,6.895 21.605,6 20.5,6 Z M5,14 L7,14 L7,10 L9,10 L9,14 L11,14 L8,17 L5,14 Z M16,10 L13,10 L13,14 L15,14 L15,12 L16,12 C16.552,12 17,11.552 17,11 C17,10.448 16.552,10 16,10 Z M19,10 L19,14 L20,14 L20,11.5 L21,11.5 L21,10 L19,10 Z" fill="#6c9cf2"/></svg>` };
const IconHTML = { template: `<svg width="16" height="16" viewBox="0 0 24 24"><path fill="#e34f26" d="M1.6 21.3l1.8-20.7h17.2l-1.8 20.7-6.8 2.7z"/><path fill="#f16529" d="M12 19.8V3.7h8.6l-1.5 17.3z"/><path fill="#ebebeb" d="M12 8.2H8.1l-.3-3.3h4.2V3.7H4.4l.6 6.9h7zM12 13H8.5l-.3 3.3 3.8 1.5v-1.2l-2.6-1h2.6z"/><path fill="#fff" d="M12 8.2v-1h3.6l.3-3.3h-3.9V2.7h7.6l-.6 6.7h-7zm0 8.1l2.6-1v1.2l-3.8 1.5v-1.2l3.8-1.5h-4.2l-.4-4.5h7.3l-.3 4.5z"/></svg>` };
const IconFile = { template: `<svg width="16" height="16" viewBox="0 0 24 24"><path d="M14,2H6A2,2 0 0,0 4,4V20A2,2 0 0,0 6,22H18A2,2 0 0,0 20,20V8L14,2M18,20H6V4H13V9H18V20Z" fill="#cccccc"/></svg>` };

const icons = {
  vue: markRaw(IconVue),
  js: markRaw(IconJS),
  json: markRaw(IconJSON),
  md: markRaw(IconMD),
  html: markRaw(IconHTML),
  file: markRaw(IconFile)
};

const getIcon = (type) => icons[type] || icons.file;

// --- STATE MANAGEMENT ---
const activeTabId = ref('README.md');
const openTabs = ref([
    { id: 'README.md', type: 'md' },
    { id: 'App.vue', type: 'vue' },
]);

const visibleSections = reactive({ open: true, files: true });
const openFolders = reactive({ src: true, components: true, public: true });

// --- MOCK DATA ---
const fileContents = {
  'App.vue': `<span class="keyword">&lt;template&gt;</span>
  <span class="keyword">&lt;div</span> <span class="attribute">id</span>=<span class="string">"app"</span><span class="keyword">&gt;</span>
    <span class="comment">&lt;!-- O seu conteúdo do portfólio vai aqui --&gt;</span>
    <span class="keyword">&lt;Desktop</span> <span class="keyword">/&gt;</span>
  <span class="keyword">&lt;/div&gt;</span>
<span class="keyword">&lt;/template&gt;</span>

<span class="keyword">&lt;script</span> <span class="attribute">setup</span><span class="keyword">&gt;</span>
<span class="keyword">import</span> <span class="variable">Desktop</span> <span class="keyword">from</span> <span class="string">'./components/Desktop.vue'</span>;
<span class="keyword">&lt;/script&gt;</span>

<span class="keyword">&lt;style&gt;</span>
  <span class="comment">/* Estilos globais */</span>
<span class="keyword">&lt;/style&gt;</span>`,
  'main.js': `<span class="keyword">import</span> { <span class="variable">createApp</span> } <span class="keyword">from</span> <span class="string">'vue'</span>
<span class="keyword">import</span> <span class="variable">App</span> <span class="keyword">from</span> <span class="string">'./App.vue'</span>

<span class="variable">createApp</span>(<span class="variable">App</span>).<span class="function">mount</span>(<span class="string">'#app'</span>)`,
  'Desktop.vue': `<span class="comment">&lt;!-- Componente Desktop --&gt;</span>
<span class="keyword">&lt;template&gt;</span>
  <span class="keyword">&lt;div</span> <span class="attribute">class</span>=<span class="string">"desktop-wrapper"</span><span class="keyword">&gt;</span>
    <span class="comment">&lt;!-- Ícones, janelas, etc. --&gt;</span>
  <span class="keyword">&lt;/div&gt;</span>
<span class="keyword">&lt;/template&gt;</span>`,
  'package.json': `{
  <span class="string">"name"</span>: <span class="string">"vue-portfolio"</span>,
  <span class="string">"version"</span>: <span class="string">"1.0.0"</span>,
  <span class="string">"private"</span>: <span class="string">true</span>,
  <span class="string">"scripts"</span>: {
    <span class="string">"dev"</span>: <span class="string">"vite"</span>,
    <span class="string">"build"</span>: <span class="string">"vite build"</span>
  },
  <span class="string">"dependencies"</span>: {
    <span class="string">"vue"</span>: <span class="string">"^3.3.0"</span>
  },
  <span class="string">"devDependencies"</span>: {
    <span class="string">"vite"</span>: <span class="string">"^4.2.0"</span>,
    <span class="string">"@vitejs/plugin-vue"</span>: <span class="string">"^4.1.0"</span>
  }
}`,
  'README.md': `<span class="markdown-header"># Portfólio Vue.js</span>
<span class="markdown-text">Um portfólio interativo inspirado numa UI de desktop moderna, construído com Vue 3 e Vite.</span>

<span class="markdown-header">## Funcionalidades</span>
<span class="markdown-text">- Interface semelhante ao VS Code</span>
<span class="markdown-text">- Janelas arrastáveis e redimensionáveis (em breve!)</span>
<span class="markdown-text">- Explorador de arquivos interativo</span>

<span class="markdown-header">## Tecnologias</span>
<span class="markdown-text">- Vue 3 (Composition API)</span>
<span class="markdown-text">- Vite</span>
<span class="markdown-text">- CSS3</span>`,
'.gitignore': `<span class="comment"># Logs</span>
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

<span class="comment"># Dependencies</span>
/node_modules
/.pnp
.pnp.js

<span class="comment"># Build output</span>
/dist
`,
'index.html': `<span class="keyword">&lt;!DOCTYPE html&gt;</span>
<span class="keyword">&lt;html</span> <span class="attribute">lang</span>=<span class="string">"en"</span><span class="keyword">&gt;</span>
  <span class="keyword">&lt;head&gt;</span>
    <span class="keyword">&lt;meta</span> <span class="attribute">charset</span>=<span class="string">"UTF-8"</span> <span class="keyword">/&gt;</span>
    <span class="keyword">&lt;link</span> <span class="attribute">rel</span>=<span class="string">"icon"</span> <span class="attribute">type</span>=<span class="string">"image/svg+xml"</span> <span class="attribute">href</span>=<span class="string">"/vite.svg"</span> <span class="keyword">/&gt;</span>
    <span class="keyword">&lt;meta</span> <span class="attribute">name</span>=<span class="string">"viewport"</span> <span class="attribute">content</span>=<span class="string">"width=device-width, initial-scale=1.0"</span> <span class="keyword">/&gt;</span>
    <span class="keyword">&lt;title&gt;</span>Vite + Vue<span class="keyword">&lt;/title&gt;</span>
  <span class="keyword">&lt;/head&gt;</span>
  <span class="keyword">&lt;body&gt;</span>
    <span class="keyword">&lt;div</span> <span class="attribute">id</span>=<span class="string">"app"</span><span class="keyword">&gt;&lt;/div&gt;</span>
    <span class="keyword">&lt;script</span> <span class="attribute">type</span>=<span class="string">"module"</span> <span class="attribute">src</span>=<span class="string">"/src/main.js"</span><span class="keyword">&gt;&lt;/script&gt;</span>
  <span class="keyword">&lt;/body&gt;</span>
<span class="keyword">&lt;/html&gt;</span>
`
};

const fileTree = [
  { name: 'public', children: [ { name: 'vite.svg', type: 'file' } ]},
  { name: 'src', children: [
    { name: 'components', children: [
      { name: 'Desktop.vue', type: 'vue' }
    ]},
    { name: 'App.vue', type: 'vue' },
    { name: 'main.js', type: 'js' },
  ]},
  { name: '.gitignore', type: 'file' },
  { name: 'index.html', type: 'html' },
  { name: 'package.json', type: 'json' },
  { name: 'README.md', type: 'md' }
];

// --- COMPUTED PROPERTIES ---
const activeTabContent = computed(() => {
    const activeTab = openTabs.value.find(tab => tab.id === activeTabId.value);
    if (!activeTab) return null;
    return {
        ...activeTab,
        content: fileContents[activeTab.id] || 'Conteúdo do arquivo não encontrado.'
    };
});

// --- METHODS ---
const setActiveTab = (tabId) => {
  activeTabId.value = tabId;
};

const openFile = (fileName, fileType) => {
    const isOpen = openTabs.value.some(tab => tab.id === fileName);
    if (!isOpen) {
        openTabs.value.push({ id: fileName, type: fileType });
    }
    setActiveTab(fileName);
};

const closeTab = (index) => {
  const closingTab = openTabs.value[index];
  openTabs.value.splice(index, 1);

  if (activeTabId.value === closingTab.id) {
    if (openTabs.value.length > 0) {
      const newIndex = Math.max(0, index - 1);
      activeTabId.value = openTabs.value[newIndex].id;
    } else {
      activeTabId.value = null;
    }
  }
};

const toggleFolder = (folderName) => {
  openFolders[folderName] = !openFolders[folderName];
};

const toggleSection = (sectionName) => {
  visibleSections[sectionName] = !visibleSections[sectionName];
};

</script>

<style scoped>
/* Main Container */
.vs-code-container {
  display: flex;
  height: 100vh;
  background-color: #1e1e1e;
  color: #d4d4d4;
  font-family: 'Segoe UI', 'Roboto', 'Helvetica Neue', Arial, sans-serif;
  font-size: 13px;
  line-height: 1.5;
}

/* Sidebar */
.sidebar {
  display: flex;
  width: 280px;
  min-width: 220px;
  background-color: #252526;
  border-right: 1px solid #3c3c3c;
}

/* Activity Bar */
.activity-bar {
  width: 48px;
  background-color: #333333;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 10px;
}
.activity-item {
  width: 100%;
  padding: 12px 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #858585;
  cursor: pointer;
  transition: color 0.2s ease;
  position: relative;
}
.activity-item:hover {
  color: #ffffff;
}
.activity-item.active {
  color: #ffffff;
}
.activity-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  height: 60%;
  width: 2px;
  background-color: #007acc;
}

/* Explorer Panel */
.explorer-panel {
  flex: 1;
  overflow-y: auto;
}
.panel-section {
    border-bottom: 1px solid #3c3c3c;
}
.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 6px 10px;
  font-size: 11px;
  font-weight: bold;
  text-transform: uppercase;
  cursor: pointer;
  user-select: none;
}
.panel-header:hover {
    background-color: #2a2d2e;
}
.panel-title {
    display: flex;
    align-items: center;
}
.panel-header .chevron {
  margin-right: 4px;
  transition: transform 0.2s ease-in-out;
}
.panel-header .chevron.rotated {
  transform: rotate(90deg);
}
.panel-actions {
    display: flex;
    align-items: center;
}
.panel-actions button {
    background: none;
    border: none;
    color: #d4d4d4;
    cursor: pointer;
    padding: 2px;
    border-radius: 3px;
}
.panel-actions button:hover {
    background-color: #4f4f53;
}


/* File & Folder Lists */
.file-list {
    padding-left: 10px;
}
.folder-content {
    padding-left: 20px;
}
.file, .folder-header {
  display: flex;
  align-items: center;
  padding: 4px 10px;
  cursor: pointer;
  border-radius: 3px;
  user-select: none;
}
.file:hover, .folder-header:hover {
  background-color: #2a2d2e;
}
.file.active {
  background-color: #37373d;
  color: #ffffff;
}
.file-icon, .folder-icon {
  margin-right: 6px;
  flex-shrink: 0;
}
.folder-icon {
    color: #d4d4d4;
}
.tree-item {
    padding-left: 5px;
}


/* Main Content Area */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* Tab Bar */
.tab-bar {
  display: flex;
  background-color: #2d2d30;
  border-bottom: 1px solid #3c3c3c;
  flex-shrink: 0;
}
.tab {
  display: flex;
  align-items: center;
  padding: 0 12px;
  height: 35px;
  background-color: #2d2d30;
  border-right: 1px solid #3c3c3c;
  cursor: pointer;
  color: #9e9e9e;
}
.tab:hover {
  background-color: #3e3e42;
}
.tab.active {
  background-color: #1e1e1e;
  color: #ffffff;
}
.tab span {
  white-space: nowrap;
  margin-left: 6px;
}
.tab-close {
  background: none;
  border: none;
  color: #9e9e9e;
  cursor: pointer;
  padding: 2px 6px;
  border-radius: 3px;
  margin-left: 8px;
  font-size: 16px;
  line-height: 1;
}
.tab-close:hover {
  background-color: #4f4f53;
  color: #ffffff;
}

/* Editor */
.editor-content {
  flex-grow: 1;
  display: flex;
  background-color: #1e1e1e;
}
.line-numbers {
  width: 50px;
  padding: 8px 0;
  text-align: right;
  color: #858585;
  font-family: 'Consolas', 'Courier New', monospace;
  user-select: none;
}
.line-number {
  padding: 0 10px;
  height: 19.5px;
  line-height: 19.5px;
}
.code-content {
  flex: 1;
  padding: 8px;
  overflow: auto;
  font-family: 'Consolas', 'Courier New', monospace;
}
.code-file {
    white-space: pre;
}
.welcome-message {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    color: #858585;
    font-size: 16px;
}

/* Status Bar */
.status-bar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #007acc;
    color: #ffffff;
    padding: 0 10px;
    height: 22px;
    font-size: 12px;
    flex-shrink: 0;
}
.status-left, .status-right {
    display: flex;
    align-items: center;
    gap: 15px;
}
.status-item {
    display: flex;
    align-items: center;
    gap: 5px;
    cursor: pointer;
}
.status-item:hover {
    background-color: rgba(255, 255, 255, 0.1);
}


/* Syntax Highlighting */
:deep(.keyword) { color: #569cd6; }
:deep(.string) { color: #ce9178; }
:deep(.comment) { color: #6a9955; }
:deep(.variable) { color: #9cdcfe; }
:deep(.function) { color: #dcdcaa; }
:deep(.attribute) { color: #9cdcfe; }
:deep(.markdown-header) { color: #569cd6; font-weight: bold; }
:deep(.markdown-text) { color: #d4d4d4; }
</style>
