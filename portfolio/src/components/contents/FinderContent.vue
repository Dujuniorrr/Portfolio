<template>
    <div class="finder-container">
    
      <div class="finder-body">
        <div 
          v-for="(column, colIndex) in columns" 
          :key="colIndex" 
          class="column"
          :class="{ 'preview-column': column.isPreview }"
        >
          <div v-if="!column.isPreview" class="column-content">
            <div 
              v-for="item in column.items" 
              :key="item.name" 
              class="finder-item"
              :class="{ 'active': isItemActive(item, colIndex) }"
              @click="handleItemClick(item, colIndex)"
            >
              <component :is="getIcon(item.type)" class="item-icon" />
              <span class="item-name">{{ item.name }}</span>
              <svg v-if="item.type === 'folder'" class="chevron" viewBox="0 0 16 16"><path fill="currentColor" d="M5.293 3.293L9.586 7.586a.5.5 0 010 .707L5.293 12.707a.5.5 0 00.707.707l4.647-4.646a.5.5 0 000-.708L6 2.586a.5.5 0 00-.707.707z"/></svg>
            </div>
          </div>
          <!-- Updated Preview Pane -->
          <div v-else class="preview-pane">
            <component :is="getIcon(column.file.type)" class="preview-icon" />
            <h2 class="preview-name">{{ column.file.name }}</h2>
            <div class="preview-details">
              <div class="detail-row">
                <span class="detail-label">Format</span>
                <span class="detail-value">{{ column.file.kind }}</span>
              </div>
              <div class="detail-row">
                <span class="detail-label">Size</span>
                <span class="detail-value">{{ column.file.size }}</span>
              </div>
               <div class="detail-row">
                <span class="detail-label">Modified</span>
                <span class="detail-value">{{ column.file.modified }}</span>
              </div>
            </div>
            <!-- Content Preview Section -->
            <div class="preview-content-container">
              <img 
                v-if="column.file.type.startsWith('image') && column.file.url" 
                :src="column.file.url" 
                :alt="column.file.name"
                class="preview-image preview"
                @error="onImageError"
              />
              <pre v-else-if="column.file.content" class="preview-text preview">{{ column.file.content }}</pre>
              <div v-else class="no-preview preview">
                <span>No Preview Available</span>
              </div>
            </div>
          </div>
        </div>
      </div>
  
      <div class="finder-footer">
        <span>{{ footerItemCount }} items</span>
        <span>128 GB available on disk</span>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, markRaw } from 'vue';
  
  // --- ICONS (as simple Vue components) ---
  const IconFolder = { template: `<svg viewBox="0 0 24 24"><path fill="#5c9ded" d="M4,6a2,2,0,0,1,2-2h4l2,2h6a2,2,0,0,1,2,2v8a2,2,0,0,1-2,2H6a2,2,0,0,1-2-2Z"/></svg>` };
  const IconImage = { template: `<svg viewBox="0 0 24 24"><path fill="#b4b4b4" d="M19,19H5V5h7l2,2h5V19M11.2,12.5l-2.7,3.5H16l-3.2-4.2l-1.9,2.5Z"/></svg>`};
  const IconText = { template: `<svg viewBox="0 0 24 24"><path fill="#b4b4b4" d="M6,2a2,2,0,0,0-2,2v16a2,2,0,0,0,2,2h12a2,2,0,0,0,2-2V8l-6-6H6m5,10H8v-2h3v2m3-2h-2v-2h2v2m-3,4H8v-2h3v2Z"/></svg>`};
  const IconCode = { template: `<svg viewBox="0 0 24 24"><path fill="#b4b4b4" d="M14.6,16.6l4.6-4.6-4.6-4.6L16,6l6,6-6,6-1.4-1.4m-5.2,0L4.8,12l4.6-4.6L8,6l-6,6,6,6,1.4-1.4Z"/></svg>`};
  
  const icons = {
    folder: markRaw(IconFolder),
    image: markRaw(IconImage),
    text: markRaw(IconText),
    code: markRaw(IconCode)
  };
  
  const getIcon = (type) => {
      if (type.startsWith('image')) return icons.image;
      if (type === 'vue' || type === 'js' || type === 'html') return icons.code;
      if (type === 'md') return icons.text;
      return icons[type] || icons.text;
  };
  
  // --- MOCK FILE SYSTEM DATA (with content) ---
  const fileSystem = ref([
    { name: 'AboutMe.md', type: 'md', kind: 'Markdown Document', size: '2 KB', modified: '2025-08-10', content: '# About Me\n\nThis is a simple markdown file to demonstrate the text preview functionality.\n\nYou can put any text content here.' },
    { name: 'Projects', type: 'folder', children: [
        { name: 'Portfolio-V1', type: 'folder', children: [
          { name: 'screenshot.png', type: 'image/png', kind: 'PNG Image', size: '1.2 MB', modified: '2025-05-20', url: 'https://placehold.co/400x300/5c9ded/white?text=Screenshot' },
          { name: 'index.html', type: 'html', kind: 'HTML Document', size: '5 KB', modified: '2025-05-19', content: '<html>\n  <body>\n    <h1>Portfolio V1</h1>\n    <p>This is the main page.</p>\n  </body>\n</html>' },
        ]},
        { name: 'Data-Visualizer', type: 'folder', children: [
          { name: 'main.js', type: 'js', kind: 'JavaScript File', size: '22 KB', modified: '2025-07-03', content: 'const app = new Vue({\n  el: \'#app\',\n  data: {\n    message: \'Hello Vue!\n  }\n});' },
          { name: 'Chart.vue', type: 'vue', kind: 'Vue Component', size: '8 KB', modified: '2025-07-02', content: '<template>\n  <div class="chart">\n    <!-- Chart content -->\n  </div>\n</template>' },
        ]},
      ]
    },
    { name: 'Contact.md', type: 'md', kind: 'Markdown Document', size: '1 KB', modified: '2025-08-12', content: '## Contact Information\n\n- Email: test@example.com\n- Phone: 555-1234' },
    { name: 'Gallery', type: 'folder', children: [
        { name: 'landscape.jpg', type: 'image/jpeg', kind: 'JPEG Image', size: '4.5 MB', modified: '2025-03-15', url: 'https://placehold.co/400x300/a8d8a0/white?text=Landscape' },
        { name: 'portrait.jpg', type: 'image/jpeg', kind: 'JPEG Image', size: '3.1 MB', modified: '2025-04-02', url: 'https://media.licdn.com/dms/image/v2/D4D03AQGdoId3hAzBlA/profile-displayphoto-crop_800_800/B4DZhc9GckHYAQ-/0/1753906179613?e=1758153600&v=beta&t=apJmaVmkcBlsd5UsHFMmG99-L2VhTARp0IrqiWf9rHQ' },
      ]
    }
  ]);
  
  // --- STATE MANAGEMENT ---
  const currentPath = ref([]);
  const selectedFile = ref(null);
  
  // --- COMPUTED PROPERTIES ---
  const columns = computed(() => {
    const result = [];
    let currentItems = fileSystem.value;
  
    // Add the root column
    result.push({ items: currentItems });
  
    // Build columns from the current path
    let parent = { children: currentItems };
    for (const segment of currentPath.value) {
      const folder = parent.children?.find(item => item.name === segment && item.type === 'folder');
      if (folder && folder.children) {
        result.push({ items: folder.children });
        parent = folder;
      } else {
        break; 
      }
    }
  
    // Add a preview column if a file is selected
    if (selectedFile.value) {
      result.push({ file: selectedFile.value, isPreview: true });
    }
  
    return result;
  });
  
  const footerItemCount = computed(() => {
      const lastColumn = columns.value[columns.value.length - 1];
      if (lastColumn.isPreview) {
          // If the last column is a preview, count items in the column before it
          if (columns.value.length > 1) {
              return columns.value[columns.value.length - 2].items.length;
          }
          return 0;
      }
      return lastColumn.items.length;
  });
  
  // --- METHODS ---
  const handleItemClick = (item, colIndex) => {
    const newPath = currentPath.value.slice(0, colIndex);
  
    if (item.type === 'folder') {
      newPath.push(item.name);
      currentPath.value = newPath;
      selectedFile.value = null; // Deselect file when navigating
    } else {
      currentPath.value = newPath;
      selectedFile.value = item;
    }
  };
  
  const isItemActive = (item, colIndex) => {
    if (item.type === 'folder') {
      return currentPath.value[colIndex] === item.name;
    }
    // A file is active if it's the currently selected file in its column
    return selectedFile.value?.name === item.name && currentPath.value.length === colIndex;
  };

  const onImageError = (event) => {
    // Replace broken image with a placeholder
    event.target.src = 'https://placehold.co/300x200/ccc/FFF?text=Image+Not+Found';
  };
  </script>
  
  
  <style scoped>
  /* Main Container */
  .finder-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    /* background-color: #f6f6f6; */
    border: 1px solid #c7c7c7;
    /* border-radius: 8px; */
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    color: #333;
    overflow: hidden;
  }
  
  /* Body */
  .finder-body {
    flex-grow: 1;
    display: flex;
    overflow-x: auto;
    background-color: #fff;
  }
  .column {
    min-width: 200px;
    border-right: 1px solid #ddd;
    overflow-y: auto;
    flex-shrink: 0; /* Prevent columns from shrinking */
  }
  .column-content {
    padding: 4px;
  }
  
  /* Items */
  .finder-item {
    display: flex;
    align-items: center;
    padding: 4px 8px;
    border-radius: 5px;
    cursor: default;
    user-select: none;
    font-size: 13px;
  }
  .finder-item:hover {
    background-color: #f0f0f0;
  }
  .finder-item.active {
    background-color: #007aff;
    color: white;
  }
  .finder-item.active .chevron,
  .finder-item.active .item-icon {
    color: white;
  }
  .item-icon {
    width: 18px;
    height: 18px;
    margin-right: 6px;
    flex-shrink: 0;
  }
  .item-name {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    flex-grow: 1;
  }
  .chevron {
    width: 16px;
    height: 16px;
    color: #a0a0a0;
    margin-left: auto;
    flex-shrink: 0;
  }
  
  /* Preview Column */
  .preview-column {
    flex-grow: 1;
    min-width: 300px; /* Increased min-width for better preview */
    padding: 20px;
    display: flex;
    flex-direction: column;
  }
  .preview-pane {
    text-align: center;
    display: flex;
    flex-direction: column;
    height: 100%;
  }
  .preview-icon {
    width: 96px;
    height: 96px;
    margin: 0 auto 16px auto; /* Center the icon */
  }
  .preview-name {
    font-size: 16px;
    font-weight: 500;
    margin: 0 0 16px 0;
    word-break: break-all;
  }
  .preview-details {
    font-size: 12px;
    color: #555;
    text-align: left;
    border-top: 1px solid #eee;
    padding-top: 8px;
    margin-top: 8px;
  }
  .detail-row {
    display: flex;
    justify-content: space-between;
    padding: 4px 0;
  }
  .detail-label {
    font-weight: 500;
    color: #888;
    padding-right: 10px;
  }
  
  /* --- NEW STYLES FOR CONTENT PREVIEW --- */
  .preview-content-container {
    margin-top: 16px;
    padding: 10px;
    
    border-radius: 6px;
    text-align: left;
    flex-grow: 1; /* Allows the container to fill available space */
    overflow-y: auto; /* Enables scrolling for large content */
    display: flex;
    flex-direction: column;
    align-items: center; /* Center content horizontally by default */
  }

  .preview {
    border: 1px solid #e0e0e0;
    background-color: #fdfdfd;
    border-radius: 6px;
    padding: 10px;
  }
  .preview-image {
    
    max-width: 100%;
    height: auto;
    border-radius: 4px;
    max-width: 100%;
    max-height: 100%;
  }

  .preview-text {
    white-space: pre-wrap;
    word-break: break-word;
    font-size: 12px;
    color: #333;
    margin: 0;
    font-family: Menlo, Monaco, Consolas, "Courier New", monospace;
    width: 100%; /* Ensure text respects container padding */
  }

  .no-preview {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: 100%;
    color: #999;
    font-size: 13px;
  }
  
  /* Footer */
  .finder-footer {
    display: flex;
    justify-content: space-between;
    padding: 4px 12px;
    height: 25px;
    background-color: #f6f6f6;
    border-top: 1px solid #c7c7c7;
    font-size: 12px;
    color: #666;
    flex-shrink: 0;
  }
  </style>
