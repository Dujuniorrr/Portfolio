<!-- Terminal.vue -->
<template>
    <div class="terminal-container" @click="focusInput">
      <div class="terminal-body" ref="terminalBody">
        <div class="terminal-line" v-for="(line, index) in output" :key="index">
          <template v-if="line.type === 'input'">
            <span class="prompt">➜</span>
            <span class="command">{{ line.text }}</span>
          </template>
          <template v-else>
            <pre class="response">{{ line.text }}</pre>
          </template>
        </div>
        <div class="terminal-line current">
          <span class="prompt">➜</span>
          <input
            ref="input"
            v-model="currentInput"
            @keyup.enter="handleCommand"
            @keydown="handleKeyUp"
            class="terminal-input"
            autofocus
          />
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, nextTick } from 'vue';
  
  const fileSystem = {
    '/': {
      type: 'dir',
      children: {
        home: {
          type: 'dir',
          children: {
            user: {
              type: 'dir',
              children: {
                projects: {
                  type: 'dir',
                  children: {
                    'portfolio-site.txt': {
                      type: 'file',
                      content: 'Meu site pessoal com Vue 3, Vite e animações.',
                    },
                    'crm-dashboard.txt': {
                      type: 'file',
                      content: 'Dashboard CRM para gestão de leads e vendas.',
                    },
                  },
                },
              },
            },
          },
        },
        etc: {
          type: 'dir',
          children: {
            'config.txt': { type: 'file', content: 'config=v1.0\nmode=dev' },
          },
        },
      },
    },
  };
  
  const currentPath = ref(['/']);
  const output = ref([
    { type: 'response', text: 'Welcome to your portfolio terminal.\nType "help" to see available commands.\n' },
  ]);
  const currentInput = ref('');
  const terminalBody = ref(null);
  const input = ref(null);
  
  const commandHistory = ref([]);
  const historyIndex = ref(-1);
  const allCommands = ['help', 'clear', 'ls', 'cd', 'pwd', 'about', 'projects', 'contact', 'cat'];
  
  const scrollToBottom = () => {
    nextTick(() => {
      terminalBody.value.scrollTop = terminalBody.value.scrollHeight;
      input.value.focus();
    });
  };
  
  const resolvePath = (pathArray) => {
    let node = fileSystem['/'];
    for (const segment of pathArray.slice(1)) {
      if (!node.children?.[segment] || node.children[segment].type !== 'dir') return null;
      node = node.children[segment];
    }
    return node;
  };
  
  const getCurrentNode = () => resolvePath(currentPath.value);
  
  const focusInput = () => input.value?.focus();
  
  const handleCommand = () => {
    const cmd = currentInput.value.trim();
    if (!cmd) return;
  
    commandHistory.value.push(cmd);
    historyIndex.value = commandHistory.value.length;
  
    output.value.push({ type: 'input', text: cmd });
  
    const [command, ...args] = cmd.split(' ');
    const param = args.join(' ');
  
    switch (command.toLowerCase()) {
      case 'help':
        output.value.push({
          type: 'response',
          text: `Available commands:
  - help: Show this message
  - clear: Clear the terminal
  - ls: List directory contents
  - cd <dir>: Change directory
  - pwd: Show current directory
  - cat <file>: Read a file
  - about: Info about me
  - projects: Show portfolio projects
  - contact: How to reach me`,
        });
        break;
  
      case 'clear':
        output.value.splice(0, output.value.length);
        break;
  
      case 'ls': {
        const node = getCurrentNode();
        if (!node) {
          output.value.push({ type: 'response', text: 'Error: Directory not found' });
        } else {
          const list = Object.keys(node.children || {}).join('\t') || '(empty)';
          output.value.push({ type: 'response', text: list });
        }
        break;
      }
  
      case 'cd': {
        if (!param) {
          output.value.push({ type: 'response', text: 'Usage: cd <directory>' });
          break;
        }
        if (param === '..') {
          if (currentPath.value.length > 1) currentPath.value.pop();
        } else {
          const newPath = [...currentPath.value, param];
          const node = resolvePath(newPath);
          if (node) {
            currentPath.value = newPath;
          } else {
            output.value.push({ type: 'response', text: `cd: no such directory: ${param}` });
          }
        }
        break;
      }
  
      case 'pwd':
        output.value.push({ type: 'response', text: currentPath.value.join('/') || '/' });
        break;
  
      case 'cat': {
        const node = getCurrentNode();
        if (!param) {
          output.value.push({ type: 'response', text: 'Usage: cat <file>' });
          break;
        }
        const file = node.children?.[param];
        if (!file || file.type !== 'file') {
          output.value.push({ type: 'response', text: `cat: ${param}: No such file` });
        } else {
          output.value.push({ type: 'response', text: file.content });
        }
        break;
      }
  
      case 'about':
        output.value.push({
          type: 'response',
          text: 'I am a web developer passionate about building creative UIs and efficient systems.',
        });
        break;
  
      case 'projects':
        output.value.push({
          type: 'response',
          text: '1. Portfolio Website\n2. CRM Dashboard\n3. AI Code Assistant',
        });
        break;
  
      case 'contact':
        output.value.push({
          type: 'response',
          text: 'Email: voce@email.com\nLinkedIn: linkedin.com/in/seuperfil',
        });
        break;
  
      default:
        output.value.push({
          type: 'response',
          text: `Command not found: ${command}\nType "help" to see available commands.`,
        });
    }
  
    currentInput.value = '';
    scrollToBottom();
  };
  
  const autocomplete = () => {
    const inputText = currentInput.value.trim();
    const [command, ...args] = inputText.split(' ');
    const partial = args.join(' ');
  
    if (args.length === 0) {
      const match = allCommands.find(c => c.startsWith(command));
      if (match) currentInput.value = match;
    } else {
      const node = getCurrentNode();
      const keys = Object.keys(node.children || {});
      const match = keys.find(k => k.startsWith(partial));
      if (match) currentInput.value = `${command} ${match}`;
    }
  };
  
  const handleKeyUp = (e) => {
    if (e.key === 'ArrowUp') {
      if (historyIndex.value > 0) {
        historyIndex.value--;
        currentInput.value = commandHistory.value[historyIndex.value];
      }
    } else if (e.key === 'ArrowDown') {
      if (historyIndex.value < commandHistory.value.length - 1) {
        historyIndex.value++;
        currentInput.value = commandHistory.value[historyIndex.value];
      } else {
        currentInput.value = '';
      }
    } else if (e.key === 'Tab') {
      e.preventDefault();
      autocomplete();
    }
  };
  
  onMounted(() => {
    scrollToBottom();
  });
  </script>
 
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500&display=swap');

.terminal-container {
  width: 100%;
  height: 100vh;
  background: linear-gradient(135deg, #1a1e2a 0%, #0d0f17 100%);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  border-radius: 8px;
  box-shadow: 
    0 0 25px rgba(0, 255, 127, 0.2),
    inset 0 0 10px rgba(0, 100, 50, 0.3);
  position: relative;
  cursor: text;
  font-family: 'Fira Code', 'Menlo', monospace;
}

.terminal-body {
  flex-grow: 1;
  padding: 40px 20px 20px;
  overflow-y: auto;
  font-size: 0.8rem;
  line-height: 1.5;
  background: rgba(10, 12, 18, 0.8);
  position: relative;
  z-index: 2;
}

.terminal-line {
  display: flex;
  align-items: flex-start;
  white-space: pre-wrap;
  margin-bottom: 8px;
  position: relative;
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(5px); }
  to { opacity: 1; transform: translateY(0); }
}

.prompt {
  color: #50fa7b;
  margin-right: 10px;
  min-width: 20px;
  text-shadow: 0 0 5px rgba(80, 250, 123, 0.5);
}

.command {
  color: #f8f8f2;
  font-weight: 500;
}

.response {
  color: #e1dde7;
  margin-left: 30px;
  width: calc(100% - 30px);
  overflow-wrap: break-word;
  text-shadow: 0 0 3px rgba(189, 147, 249, 0.3);
}

.terminal-line.current {
  position: relative;
}

.terminal-input {
  background: transparent;
  border: none;
  color: #f8f8f2;
  outline: none;
  font-family: inherit;
  flex-grow: 1;
  font-size: 0.95rem;
  padding: 0;
  caret-color: #50fa7b;
  caret-shape: block;
  text-shadow: 0 0 3px rgba(248, 248, 242, 0.3);
}

.terminal-input::selection {
  background: rgba(80, 250, 123, 0.3);
}

/* Efeito de linha de digitação */
.terminal-line.current::after {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  bottom: -3px;
  height: 1px;
  /* background: linear-gradient(90deg, 
    transparent 0%, 
    #50fa7b 30%, 
    #50fa7b 70%, 
    transparent 100%); */
  opacity: 0.7;
}

/* Scrollbar estilizada */
.terminal-body::-webkit-scrollbar {
  width: 8px;
}

.terminal-body::-webkit-scrollbar-track {
  background: rgba(10, 15, 25, 0.5);
}

.terminal-body::-webkit-scrollbar-thumb {
  background: rgba(80, 250, 123, 0.3);
  border-radius: 4px;
}

.terminal-body::-webkit-scrollbar-thumb:hover {
  background: rgba(80, 250, 123, 0.5);
}
</style>