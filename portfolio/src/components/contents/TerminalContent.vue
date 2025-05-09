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
  .terminal-container {
    width: 100%;
    height: 100vh;
    color: #00ff7f;
    font-family: 'Menlo', monospace;
    display: flex;
    justify-content: center;
    background-color: #1e1e1e;
    overflow: hidden;
    flex-direction: column;
    box-shadow: 0 0 10px rgba(0, 255, 127, 0.3);
    cursor: text;
  }
  .terminal-body {
    flex-grow: 1;
    padding: 15px;
    overflow-y: auto;
    font-size: 0.9em;
  }
  .terminal-line {
    display: flex;
    align-items: flex-start;
    white-space: pre-wrap;
    margin-bottom: 4px;
  }
  .prompt {
    color: #00ff7f;
    margin-right: 5px;
  }
  .command {
    color: #ffffff;
  }
  .response {
    margin-left: 1.5em;
    color: #cccccc;
  }
  .terminal-input {
    margin-top: 0.55px;
    background: none;
    border: none;
    color: #00ff7f;
    outline: none;
    font-family: inherit;
    flex-grow: 1;
  }
  </style>
  