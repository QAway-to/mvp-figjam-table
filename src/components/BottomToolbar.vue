<template>
  <div class="bottom-toolbar">
    <div class="toolbar-content">
      <!-- Cursor -->
      <button 
        class="tool-btn" 
        :class="{ active: currentTool === 'cursor' }"
        @click="$emit('tool-change', 'cursor')"
        title="Select"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M5 3L19 12L12 13L9 20L5 3Z" stroke="currentColor" stroke-width="2" stroke-linejoin="round"/>
        </svg>
      </button>
      
      <!-- Hand (Pan) -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'hand' }"
        @click="$emit('tool-change', 'hand')"
        title="Pan"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M18 11V6C18 5.45 17.55 5 17 5C16.45 5 16 5.45 16 6V11M16 9V4C16 3.45 15.55 3 15 3C14.45 3 14 3.45 14 4V9M14 8V5C14 4.45 13.55 4 13 4C12.45 4 12 4.45 12 5V12M18 11C18 11 20 11.5 20 14V17C20 19.21 18.21 21 16 21H12C10.9 21 9.85 20.58 9.05 19.85L4.5 15.5C4.19 15.22 4 14.83 4 14.42C4 13.63 4.63 13 5.42 13H5.5C5.87 13 6.22 13.14 6.5 13.38L8 14.65" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
      
      <div class="separator"></div>
      
      <!-- Marker -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'marker' }"
        @click="$emit('tool-change', 'marker')"
        title="Marker"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M15.5 5L19 8.5L8.5 19H5V15.5L15.5 5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M13 7.5L16.5 11" stroke="currentColor" stroke-width="1.5"/>
        </svg>
      </button>
      
      <!-- Highlighter -->
      <button 
        class="tool-btn highlighter"
        :class="{ active: currentTool === 'highlighter' }"
        @click="$emit('tool-change', 'highlighter')"
        title="Highlighter"
      >
        <div class="highlighter-icon"></div>
      </button>
      
      <div class="separator"></div>
      
      <!-- Shapes -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'shape' }"
        @click="$emit('tool-change', 'shape')"
        title="Shapes"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <rect x="4" y="4" width="16" height="16" rx="2" stroke="currentColor" stroke-width="1.5"/>
        </svg>
      </button>
      
      <!-- Line -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'line' }"
        @click="$emit('tool-change', 'line')"
        title="Connector"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M5 12C5 12 9 5 12 5C15 5 19 12 19 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
      </button>
      
      <div class="separator"></div>
      
      <!-- Text -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'text' }"
        @click="$emit('tool-change', 'text')"
        title="Text"
      >
        <span class="text-icon">T</span>
      </button>
      
      <!-- Sticky Note -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'sticky' }"
        @click="$emit('tool-change', 'sticky')"
        title="Sticky Note"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <rect x="4" y="4" width="16" height="16" rx="2" fill="#FEE75C" stroke="#E5C800" stroke-width="1"/>
          <path d="M16 4V8C16 9.1 16.9 10 18 10H20" stroke="#E5C800" stroke-width="1"/>
        </svg>
      </button>
      
      <!-- Table (highlighted) -->
      <button 
        class="tool-btn table-btn"
        :class="{ active: currentTool === 'table' }"
        @click="$emit('tool-change', 'table')"
        title="Table"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
          <rect x="3" y="3" width="18" height="18" rx="2" stroke="currentColor" stroke-width="1.5" fill="none"/>
          <line x1="3" y1="9" x2="21" y2="9" stroke="currentColor" stroke-width="1.5"/>
          <line x1="3" y1="15" x2="21" y2="15" stroke="currentColor" stroke-width="1.5"/>
          <line x1="9" y1="3" x2="9" y2="21" stroke="currentColor" stroke-width="1.5"/>
          <line x1="15" y1="3" x2="15" y2="21" stroke="currentColor" stroke-width="1.5"/>
        </svg>
      </button>
      
      <!-- Person/Stamp -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'stamp' }"
        @click="$emit('tool-change', 'stamp')"
        title="Stamps"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <circle cx="12" cy="8" r="4" stroke="currentColor" stroke-width="1.5"/>
          <path d="M5 20C5 16.13 8.13 13 12 13C15.87 13 19 16.13 19 20" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
      </button>
      
      <!-- Comment -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'comment' }"
        @click="$emit('tool-change', 'comment')"
        title="Comment"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <path d="M21 11.5C21 16.19 16.97 20 12 20C10.37 20 8.83 19.58 7.5 18.84L3 20L4.34 16.19C3.49 14.81 3 13.21 3 11.5C3 6.81 7.03 3 12 3C16.97 3 21 6.81 21 11.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
        </svg>
      </button>
      
      <!-- Widgets/Plugins -->
      <button 
        class="tool-btn"
        :class="{ active: currentTool === 'widgets' }"
        @click="$emit('tool-change', 'widgets')"
        title="Widgets"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
          <circle cx="6" cy="6" r="2.5" stroke="currentColor" stroke-width="1.5"/>
          <circle cx="18" cy="6" r="2.5" stroke="currentColor" stroke-width="1.5"/>
          <circle cx="6" cy="18" r="2.5" stroke="currentColor" stroke-width="1.5"/>
          <circle cx="18" cy="18" r="2.5" stroke="currentColor" stroke-width="1.5"/>
          <path d="M6 8.5V15.5M8.5 6H15.5M18 8.5V15.5M8.5 18H15.5" stroke="currentColor" stroke-width="1.2"/>
        </svg>
      </button>
      
      <div class="separator"></div>
      
      <!-- Plus -->
      <button class="tool-btn" title="More tools">
        <span class="plus-icon">+</span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BottomToolbar',
  props: {
    currentTool: {
      type: String,
      default: 'cursor'
    }
  },
  emits: ['tool-change']
}
</script>

<style scoped>
.bottom-toolbar {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1000;
}

.toolbar-content {
  display: flex;
  align-items: center;
  gap: 4px;
  background: #2c2c2c;
  border-radius: 12px;
  padding: 8px 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.tool-btn {
  width: 40px;
  height: 40px;
  border: none;
  background: transparent;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #9e9e9e;
  transition: all 0.15s ease;
}

.tool-btn:hover {
  background: #3c3c3c;
  color: #fff;
}

.tool-btn.active {
  background: #7c4dff;
  color: #fff;
}

.tool-btn.table-btn.active {
  background: #9c27b0;
}

.separator {
  width: 1px;
  height: 24px;
  background: #444;
  margin: 0 4px;
}

.text-icon {
  font-size: 18px;
  font-weight: 600;
  font-family: serif;
}

.plus-icon {
  font-size: 22px;
  font-weight: 300;
}

.highlighter-icon {
  width: 20px;
  height: 24px;
  background: linear-gradient(to bottom, #f8a5a5 0%, #f8a5a5 70%, #d47878 70%, #d47878 100%);
  border-radius: 3px;
  transform: skewX(-5deg);
}
</style>
