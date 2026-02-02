<template>
  <div class="top-toolbar" :style="toolbarStyle">
    <div class="toolbar-content">
      <!-- Color Picker -->
      <div class="tool-group">
        <button class="tool-btn color-btn" @click="toggleColorPicker">
          <div class="color-circle" :style="{ background: currentColor }"></div>
          <svg class="dropdown-arrow" width="8" height="8" viewBox="0 0 8 8">
            <path d="M1 2L4 5L7 2" stroke="currentColor" stroke-width="1.5" fill="none"/>
          </svg>
        </button>
        <div v-if="showColorPicker" class="color-picker-dropdown">
          <div 
            v-for="color in colors" 
            :key="color"
            class="color-option"
            :style="{ background: color }"
            @click="selectColor(color)"
          ></div>
        </div>
      </div>
      
      <div class="separator"></div>
      
      <!-- Font Selector -->
      <button class="tool-btn font-btn" @click="toggleFontMenu">
        <span class="font-label">Aa</span>
        <svg class="dropdown-arrow" width="8" height="8" viewBox="0 0 8 8">
          <path d="M1 2L4 5L7 2" stroke="currentColor" stroke-width="1.5" fill="none"/>
        </svg>
      </button>
      
      <!-- Size Selector -->
      <div class="tool-group">
        <button class="tool-btn size-btn" @click="toggleSizeMenu">
          <span>{{ currentSizeLabel }}</span>
          <svg class="dropdown-arrow" width="8" height="8" viewBox="0 0 8 8">
            <path d="M1 2L4 5L7 2" stroke="currentColor" stroke-width="1.5" fill="none"/>
          </svg>
        </button>
        <div v-if="showSizeMenu" class="dropdown-menu">
          <div 
            v-for="size in sizes" 
            :key="size.value"
            class="menu-item"
            :class="{ active: currentSize === size.value }"
            @click="selectSize(size.value)"
          >
            {{ size.label }}
          </div>
        </div>
      </div>
      
      <div class="separator"></div>
      
      <!-- Bold -->
      <button 
        class="tool-btn format-btn"
        :class="{ active: isBold }"
        @click="toggleBold"
        title="Bold"
      >
        <strong>B</strong>
      </button>
      
      <!-- Strikethrough -->
      <button 
        class="tool-btn format-btn"
        :class="{ active: isStrikethrough }"
        @click="toggleStrikethrough"
        title="Strikethrough"
      >
        <s>S</s>
      </button>
      
      <!-- List -->
      <button 
        class="tool-btn format-btn"
        :class="{ active: isList }"
        @click="toggleList"
        title="List"
      >
        <svg width="16" height="16" viewBox="0 0 16 16">
          <circle cx="2" cy="4" r="1.5" fill="currentColor"/>
          <circle cx="2" cy="8" r="1.5" fill="currentColor"/>
          <circle cx="2" cy="12" r="1.5" fill="currentColor"/>
          <line x1="6" y1="4" x2="14" y2="4" stroke="currentColor" stroke-width="1.5"/>
          <line x1="6" y1="8" x2="14" y2="8" stroke="currentColor" stroke-width="1.5"/>
          <line x1="6" y1="12" x2="14" y2="12" stroke="currentColor" stroke-width="1.5"/>
        </svg>
      </button>
      
      <!-- Alignment -->
      <div class="tool-group">
        <button class="tool-btn format-btn" @click="toggleAlignMenu">
          <svg width="16" height="16" viewBox="0 0 16 16">
            <line x1="1" y1="3" x2="15" y2="3" stroke="currentColor" stroke-width="1.5"/>
            <line x1="1" y1="7" x2="11" y2="7" stroke="currentColor" stroke-width="1.5"/>
            <line x1="1" y1="11" x2="15" y2="11" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          <svg class="dropdown-arrow" width="8" height="8" viewBox="0 0 8 8">
            <path d="M1 2L4 5L7 2" stroke="currentColor" stroke-width="1.5" fill="none"/>
          </svg>
        </button>
        <div v-if="showAlignMenu" class="dropdown-menu">
          <div class="menu-item" @click="setAlign('left')">Left</div>
          <div class="menu-item" @click="setAlign('center')">Center</div>
          <div class="menu-item" @click="setAlign('right')">Right</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TopToolbar',
  props: {
    table: {
      type: Object,
      required: true
    }
  },
  emits: ['update-style'],
  data() {
    return {
      showColorPicker: false,
      showSizeMenu: false,
      showAlignMenu: false,
      colors: [
        '#1e1e1e', '#616161', '#9e9e9e',
        '#f44336', '#e91e63', '#9c27b0',
        '#673ab7', '#3f51b5', '#2196f3',
        '#03a9f4', '#00bcd4', '#009688',
        '#4caf50', '#8bc34a', '#cddc39',
        '#ffeb3b', '#ffc107', '#ff9800'
      ],
      sizes: [
        { label: 'Small', value: 12 },
        { label: 'Medium', value: 16 },
        { label: 'Large', value: 20 },
        { label: 'XL', value: 28 }
      ]
    }
  },
  computed: {
    toolbarStyle() {
      if (!this.table) return {}
      // Position above the table
      return {
        left: `${this.table.x + (this.table.cols * this.table.cellWidth) / 2}px`,
        top: `${this.table.y - 60}px`
      }
    },
    currentColor() {
      return this.table?.style?.fill || '#1e1e1e'
    },
    currentSize() {
      return this.table?.style?.fontSize || 14
    },
    currentSizeLabel() {
      const size = this.sizes.find(s => s.value === this.currentSize)
      return size?.label || 'Small'
    },
    isBold() {
      return this.table?.style?.fontStyle === 'bold'
    },
    isStrikethrough() {
      return this.table?.style?.textDecoration === 'line-through'
    },
    isList() {
      return false // TODO: implement
    }
  },
  methods: {
    toggleColorPicker() {
      this.showColorPicker = !this.showColorPicker
      this.showSizeMenu = false
      this.showAlignMenu = false
    },
    toggleFontMenu() {
      // TODO: implement font selection
    },
    toggleSizeMenu() {
      this.showSizeMenu = !this.showSizeMenu
      this.showColorPicker = false
      this.showAlignMenu = false
    },
    toggleAlignMenu() {
      this.showAlignMenu = !this.showAlignMenu
      this.showColorPicker = false
      this.showSizeMenu = false
    },
    selectColor(color) {
      this.$emit('update-style', { fill: color })
      this.showColorPicker = false
    },
    selectSize(size) {
      this.$emit('update-style', { fontSize: size })
      this.showSizeMenu = false
    },
    toggleBold() {
      const newStyle = this.isBold ? 'normal' : 'bold'
      this.$emit('update-style', { fontStyle: newStyle })
    },
    toggleStrikethrough() {
      const newDecoration = this.isStrikethrough ? '' : 'line-through'
      this.$emit('update-style', { textDecoration: newDecoration })
    },
    toggleList() {
      // TODO: implement list toggle
    },
    setAlign(align) {
      this.$emit('update-style', { align })
      this.showAlignMenu = false
    }
  }
}
</script>

<style scoped>
.top-toolbar {
  position: absolute;
  transform: translateX(-50%);
  z-index: 1000;
}

.toolbar-content {
  display: flex;
  align-items: center;
  gap: 2px;
  background: #2c2c2c;
  border-radius: 8px;
  padding: 6px 10px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.25);
}

.tool-btn {
  height: 32px;
  min-width: 32px;
  border: none;
  background: transparent;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  color: #e0e0e0;
  font-size: 14px;
  padding: 0 8px;
  transition: all 0.15s ease;
}

.tool-btn:hover {
  background: #3c3c3c;
}

.tool-btn.active {
  background: #4a4a4a;
  color: #fff;
}

.color-btn {
  padding: 0 6px;
}

.color-circle {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid #555;
}

.dropdown-arrow {
  margin-left: 2px;
}

.separator {
  width: 1px;
  height: 20px;
  background: #444;
  margin: 0 6px;
}

.font-label {
  font-size: 15px;
  font-weight: 500;
}

.size-btn {
  min-width: 70px;
}

.format-btn {
  font-size: 15px;
  padding: 0 10px;
}

.format-btn strong {
  font-weight: 700;
}

.format-btn s {
  text-decoration: line-through;
}

.tool-group {
  position: relative;
}

.color-picker-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: 8px;
  background: #2c2c2c;
  border-radius: 8px;
  padding: 8px;
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 6px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
}

.color-option {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  cursor: pointer;
  transition: transform 0.15s ease;
}

.color-option:hover {
  transform: scale(1.2);
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: 8px;
  background: #2c2c2c;
  border-radius: 8px;
  padding: 4px 0;
  min-width: 100px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
}

.menu-item {
  padding: 8px 16px;
  cursor: pointer;
  color: #e0e0e0;
  font-size: 13px;
}

.menu-item:hover {
  background: #3c3c3c;
}

.menu-item.active {
  background: #4a4a4a;
}
</style>
