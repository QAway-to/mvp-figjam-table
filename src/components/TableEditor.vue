<template>
  <div class="table-editor" ref="editorContainer">
    <v-stage
      ref="stage"
      :config="stageConfig"
      @mousedown="onStageMouseDown"
      @mousemove="onStageMouseMove"
      @mouseup="onStageMouseUp"
      @contextmenu="onContextMenu"
    >
      <!-- Background Layer with dots -->
      <v-layer ref="backgroundLayer">
        <v-rect :config="backgroundConfig" />
        <!-- Dot pattern -->
        <v-circle
          v-for="dot in dots"
          :key="dot.id"
          :config="dot"
        />
      </v-layer>
      
      <!-- Tables Layer -->
      <v-layer ref="tablesLayer">
        <v-group
          v-for="table in tables"
          :key="table.id"
          :config="getTableGroupConfig(table)"
          @dragend="onTableDragEnd($event, table)"
          @click="onTableClick(table)"
        >
          <!-- Table background -->
          <v-rect :config="getTableBackgroundConfig(table)" />
          
          <!-- Cells -->
          <template v-for="(row, rowIndex) in table.cells">
            <v-group
              v-for="(cell, colIndex) in row"
              :key="`cell-${rowIndex}-${colIndex}`"
            >
              <!-- Cell background -->
              <v-rect :config="getCellConfig(table, rowIndex, colIndex)" 
                @click="onCellClick($event, table, rowIndex, colIndex)"
                @dblclick="onCellDoubleClick($event, table, rowIndex, colIndex)"
              />
              <!-- Cell text -->
              <v-text :config="getCellTextConfig(table, cell, rowIndex, colIndex)" />
            </v-group>
          </template>
          
          <!-- Selection UI (when selected) -->
          <template v-if="selectedTableId === table.id">
            <!-- Selection border -->
            <v-rect :config="getSelectionBorderConfig(table)" />
            
            <!-- Corner handles -->
            <v-rect
              v-for="handle in getCornerHandles(table)"
              :key="handle.id"
              :config="handle"
            />
            
            <!-- Column highlight (when hovering add column button) -->
            <v-rect 
              v-if="isHoveringColumnAdd"
              :config="getColumnEdgeHighlightConfig(table)"
            />
            
            <!-- Row highlight (when hovering add row button) -->
            <v-rect 
              v-if="isHoveringRowAdd"
              :config="getRowEdgeHighlightConfig(table)"
            />
            
            <!-- Column add button -->
            <v-group 
              :config="getColumnAddButtonConfig(table)" 
              @click="addColumn(table)"
              @mouseenter="isHoveringColumnAdd = true"
              @mouseleave="isHoveringColumnAdd = false"
            >
              <v-rect :config="getColumnAddBgConfig()" />
              <v-text :config="columnAddTextConfig" />
            </v-group>
            
            <!-- Row add button -->
            <v-group 
              :config="getRowAddButtonConfig(table)" 
              @click="addRow(table)"
              @mouseenter="isHoveringRowAdd = true"
              @mouseleave="isHoveringRowAdd = false"
            >
              <v-rect :config="getRowAddBgConfig(table)" />
              <v-text :config="getRowAddTextConfig(table)" />
            </v-group>
            
            <!-- Column highlight (when hovering) -->
            <v-rect 
              v-if="hoveredColumn !== null"
              :config="getColumnHighlightConfig(table, hoveredColumn)"
            />
          </template>
        </v-group>
        
        <!-- Table creation preview -->
        <v-group v-if="isCreatingTable" :config="{ x: creationPreview.x, y: creationPreview.y }">
          <v-rect :config="creationPreviewConfig" />
          <v-text :config="creationPreviewTextConfig" />
        </v-group>
      </v-layer>
    </v-stage>
    
    <!-- Text input overlay -->
    <textarea
      v-if="editingCell"
      ref="cellInput"
      class="cell-input"
      :style="cellInputStyle"
      v-model="editingCellText"
      @blur="finishCellEdit"
      @keydown="onCellInputKeydown"
    />
  </div>
</template>

<script>
export default {
  name: 'TableEditor',
  props: {
    tables: {
      type: Array,
      default: () => []
    },
    selectedTableId: {
      type: Number,
      default: null
    },
    currentTool: {
      type: String,
      default: 'cursor'
    }
  },
  emits: ['table-selected', 'table-updated', 'create-table'],
  data() {
    return {
      stageWidth: 800,
      stageHeight: 600,
      hoveredColumn: null,
      hoveredRow: null,
      isHoveringColumnAdd: false,
      isHoveringRowAdd: false,
      editingCell: null,
      editingCellText: '',
      isCreatingTable: false,
      creationPreview: { x: 0, y: 0, rows: 2, cols: 2 },
      isDraggingCreation: false,
      creationStart: { x: 0, y: 0 }
    }
  },
  computed: {
    stageConfig() {
      return {
        width: this.stageWidth,
        height: this.stageHeight
      }
    },
    backgroundConfig() {
      return {
        x: 0,
        y: 0,
        width: this.stageWidth,
        height: this.stageHeight,
        fill: '#f5f5f5'
      }
    },
    dots() {
      const dots = []
      const spacing = 20
      let id = 0
      for (let x = spacing; x < this.stageWidth; x += spacing) {
        for (let y = spacing; y < this.stageHeight; y += spacing) {
          dots.push({
            id: id++,
            x,
            y,
            radius: 1.5,
            fill: '#d0d0d0'
          })
        }
      }
      return dots
    },
    creationPreviewConfig() {
      return {
        width: this.creationPreview.cols * 100,
        height: this.creationPreview.rows * 40,
        fill: '#e8e8e8',
        stroke: '#ccc',
        strokeWidth: 1,
        cornerRadius: 8,
        opacity: 0.8
      }
    },
    creationPreviewTextConfig() {
      return {
        text: 'Click and drag to set size',
        x: 0,
        y: this.creationPreview.rows * 40 + 10,
        width: this.creationPreview.cols * 100,
        align: 'center',
        fontSize: 12,
        fill: '#666'
      }
    },
    columnAddTextConfig() {
      const table = this.tables.find(t => t.id === this.selectedTableId)
      const height = table ? table.rows * table.cellHeight : 80
      return {
        text: '+',
        x: 0,
        y: height / 2 - 10,
        width: 24,
        align: 'center',
        fontSize: 18,
        fill: this.isHoveringColumnAdd ? '#ffffff' : '#1976d2'
      }
    },
    cellInputStyle() {
      if (!this.editingCell) return {}
      const { table, rowIndex, colIndex } = this.editingCell
      const stage = this.$refs.stage?.getStage()
      if (!stage) return {}
      
      const container = this.$refs.editorContainer
      const rect = container.getBoundingClientRect()
      
      return {
        left: `${table.x + colIndex * table.cellWidth}px`,
        top: `${table.y + rowIndex * table.cellHeight}px`,
        width: `${table.cellWidth}px`,
        height: `${table.cellHeight}px`,
        fontSize: `${table.style?.fontSize || 14}px`,
        fontFamily: table.style?.fontFamily || 'Inter, sans-serif'
      }
    }
  },
  mounted() {
    this.updateStageSize()
    window.addEventListener('resize', this.updateStageSize)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.updateStageSize)
  },
  methods: {
    updateStageSize() {
      const container = this.$refs.editorContainer
      if (container) {
        this.stageWidth = container.clientWidth
        this.stageHeight = container.clientHeight
      }
    },
    
    getTableGroupConfig(table) {
      return {
        x: table.x,
        y: table.y,
        draggable: this.currentTool === 'cursor' && this.selectedTableId === table.id
      }
    },
    
    getTableBackgroundConfig(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      return {
        width,
        height,
        fill: '#ffffff',
        cornerRadius: 4,
        shadowColor: '#000',
        shadowBlur: 8,
        shadowOpacity: 0.1,
        shadowOffset: { x: 0, y: 2 }
      }
    },
    
    getCellConfig(table, rowIndex, colIndex) {
      return {
        x: colIndex * table.cellWidth,
        y: rowIndex * table.cellHeight,
        width: table.cellWidth,
        height: table.cellHeight,
        fill: '#ffffff',
        stroke: '#e0e0e0',
        strokeWidth: 1
      }
    },
    
    getCellTextConfig(table, cell, rowIndex, colIndex) {
      const style = table.style || {}
      return {
        x: colIndex * table.cellWidth + 8,
        y: rowIndex * table.cellHeight + 8,
        width: table.cellWidth - 16,
        height: table.cellHeight - 16,
        text: cell.text || '',
        fontSize: style.fontSize || 14,
        fontFamily: style.fontFamily || 'Inter, sans-serif',
        fontStyle: style.fontStyle || 'normal',
        textDecoration: style.textDecoration || '',
        fill: style.fill || '#1e1e1e',
        align: style.align || 'left',
        verticalAlign: 'middle'
      }
    },
    
    getSelectionBorderConfig(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      return {
        x: -2,
        y: -2,
        width: width + 4,
        height: height + 4,
        stroke: '#0d99ff',
        strokeWidth: 2,
        fill: 'transparent',
        cornerRadius: 4
      }
    },
    
    getCornerHandles(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      const size = 10
      const offset = -5
      
      return [
        { id: 'tl', x: offset, y: offset, width: size, height: size, fill: '#fff', stroke: '#0d99ff', strokeWidth: 2, cornerRadius: 2 },
        { id: 'tr', x: width + offset, y: offset, width: size, height: size, fill: '#fff', stroke: '#0d99ff', strokeWidth: 2, cornerRadius: 2 },
        { id: 'bl', x: offset, y: height + offset, width: size, height: size, fill: '#fff', stroke: '#0d99ff', strokeWidth: 2, cornerRadius: 2 },
        { id: 'br', x: width + offset, y: height + offset, width: size, height: size, fill: '#fff', stroke: '#0d99ff', strokeWidth: 2, cornerRadius: 2 }
      ]
    },
    
    getColumnAddButtonConfig(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      return {
        x: width + 8,
        y: 0
      }
    },
    
    getColumnAddBgConfig() {
      const table = this.tables.find(t => t.id === this.selectedTableId)
      if (!table) return {}
      const height = table.rows * table.cellHeight
      return {
        width: 24,
        height: height,
        fill: this.isHoveringColumnAdd ? '#0d99ff' : '#e3f2fd',
        cornerRadius: 4
      }
    },
    
    getRowAddButtonConfig(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      return {
        x: 0,
        y: height + 8
      }
    },
    
    getRowAddBgConfig(table) {
      const width = table.cols * table.cellWidth
      return {
        width: width,
        height: 24,
        fill: this.isHoveringRowAdd ? '#0d99ff' : '#e3f2fd',
        cornerRadius: 4
      }
    },
    
    getRowAddTextConfig(table) {
      const width = table.cols * table.cellWidth
      return {
        text: '+',
        x: 0,
        y: 2,
        width: width,
        align: 'center',
        fontSize: 16,
        fill: this.isHoveringRowAdd ? '#ffffff' : '#1976d2'
      }
    },
    
    getColumnEdgeHighlightConfig(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      return {
        x: width - 4,
        y: 0,
        width: 8,
        height: height,
        fill: '#0d99ff',
        cornerRadius: 2
      }
    },
    
    getRowEdgeHighlightConfig(table) {
      const width = table.cols * table.cellWidth
      const height = table.rows * table.cellHeight
      return {
        x: 0,
        y: height - 4,
        width: width,
        height: 8,
        fill: '#0d99ff',
        cornerRadius: 2
      }
    },
    
    getColumnHighlightConfig(table, colIndex) {
      return {
        x: colIndex * table.cellWidth,
        y: -8,
        width: table.cellWidth,
        height: table.rows * table.cellHeight + 16,
        fill: '#0d99ff',
        opacity: 0.2,
        cornerRadius: 4
      }
    },
    
    onStageMouseDown(e) {
      if (this.currentTool === 'table') {
        const pos = e.target.getStage().getPointerPosition()
        this.creationStart = { x: pos.x, y: pos.y }
        this.creationPreview = { x: pos.x, y: pos.y, rows: 2, cols: 2 }
        this.isCreatingTable = true
        this.isDraggingCreation = true
      } else if (this.currentTool === 'cursor') {
        // Check if clicked on empty space
        if (e.target === e.target.getStage() || e.target.name() === 'background') {
          this.$emit('table-selected', null)
        }
      }
    },
    
    onStageMouseMove(e) {
      if (this.isDraggingCreation && this.currentTool === 'table') {
        const pos = e.target.getStage().getPointerPosition()
        const deltaX = pos.x - this.creationStart.x
        const deltaY = pos.y - this.creationStart.y
        
        // Calculate rows and cols based on drag
        const cols = Math.max(1, Math.min(6, Math.ceil(Math.abs(deltaX) / 100)))
        const rows = Math.max(1, Math.min(8, Math.ceil(Math.abs(deltaY) / 40)))
        
        this.creationPreview.cols = cols
        this.creationPreview.rows = rows
      }
    },
    
    onStageMouseUp(e) {
      if (this.isDraggingCreation && this.currentTool === 'table') {
        this.$emit('create-table', {
          x: this.creationPreview.x,
          y: this.creationPreview.y,
          rows: this.creationPreview.rows,
          cols: this.creationPreview.cols
        })
        this.isCreatingTable = false
        this.isDraggingCreation = false
      }
    },
    
    onContextMenu(e) {
      e.evt.preventDefault()
      // Context menu logic here
    },
    
    onTableClick(table) {
      this.$emit('table-selected', table.id)
    },
    
    onTableDragEnd(e, table) {
      const updated = { ...table }
      updated.x = e.target.x()
      updated.y = e.target.y()
      this.$emit('table-updated', updated)
    },
    
    onCellClick(e, table, rowIndex, colIndex) {
      e.cancelBubble = true
      this.$emit('table-selected', table.id)
    },
    
    onCellDoubleClick(e, table, rowIndex, colIndex) {
      e.cancelBubble = true
      this.editingCell = { table, rowIndex, colIndex }
      this.editingCellText = table.cells[rowIndex][colIndex].text || ''
      this.$nextTick(() => {
        this.$refs.cellInput?.focus()
      })
    },
    
    finishCellEdit() {
      if (!this.editingCell) return
      
      const { table, rowIndex, colIndex } = this.editingCell
      const updated = JSON.parse(JSON.stringify(table))
      updated.cells[rowIndex][colIndex].text = this.editingCellText
      this.$emit('table-updated', updated)
      
      this.editingCell = null
      this.editingCellText = ''
    },
    
    onCellInputKeydown(e) {
      if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault()
        this.finishCellEdit()
      } else if (e.key === 'Tab') {
        e.preventDefault()
        this.moveToNextCell(e.shiftKey)
      } else if (e.key === 'Escape') {
        this.editingCell = null
        this.editingCellText = ''
      }
    },
    
    moveToNextCell(reverse = false) {
      if (!this.editingCell) return
      
      const { table, rowIndex, colIndex } = this.editingCell
      this.finishCellEdit()
      
      let nextRow = rowIndex
      let nextCol = colIndex + (reverse ? -1 : 1)
      
      if (nextCol >= table.cols) {
        nextCol = 0
        nextRow++
      } else if (nextCol < 0) {
        nextCol = table.cols - 1
        nextRow--
      }
      
      if (nextRow >= 0 && nextRow < table.rows) {
        this.editingCell = { table, rowIndex: nextRow, colIndex: nextCol }
        this.editingCellText = table.cells[nextRow][nextCol].text || ''
        this.$nextTick(() => {
          this.$refs.cellInput?.focus()
        })
      }
    },
    
    addColumn(table) {
      const updated = JSON.parse(JSON.stringify(table))
      updated.cols++
      updated.cells.forEach(row => {
        row.push({ text: '' })
      })
      this.$emit('table-updated', updated)
    },
    
    addRow(table) {
      const updated = JSON.parse(JSON.stringify(table))
      updated.rows++
      const newRow = []
      for (let i = 0; i < updated.cols; i++) {
        newRow.push({ text: '' })
      }
      updated.cells.push(newRow)
      this.$emit('table-updated', updated)
    }
  }
}
</script>

<style scoped>
.table-editor {
  width: 100%;
  height: 100%;
  position: relative;
}

.cell-input {
  position: absolute;
  border: 2px solid #0d99ff;
  border-radius: 2px;
  padding: 8px;
  outline: none;
  resize: none;
  background: white;
  z-index: 100;
}
</style>
