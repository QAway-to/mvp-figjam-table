<template>
  <div class="app-container">
    <!-- Top Toolbar (shows when table selected) -->
    <TopToolbar 
      v-if="selectedTable"
      :table="selectedTable"
      @update-style="updateTableStyle"
    />
    
    <!-- Main Canvas Area -->
    <div class="canvas-container" ref="canvasContainer">
      <TableEditor
        ref="tableEditor"
        :tables="tables"
        :selected-table-id="selectedTableId"
        :current-tool="currentTool"
        @table-selected="onTableSelected"
        @table-updated="onTableUpdated"
        @create-table="onCreateTable"
      />
    </div>
    
    <!-- Bottom Toolbar -->
    <BottomToolbar
      :current-tool="currentTool"
      @tool-change="onToolChange"
    />
  </div>
</template>

<script>
import TableEditor from './components/TableEditor.vue'
import BottomToolbar from './components/BottomToolbar.vue'
import TopToolbar from './components/TopToolbar.vue'

export default {
  name: 'App',
  components: {
    TableEditor,
    BottomToolbar,
    TopToolbar
  },
  data() {
    return {
      currentTool: 'cursor',
      tables: [],
      selectedTableId: null,
      nextTableId: 1
    }
  },
  computed: {
    selectedTable() {
      return this.tables.find(t => t.id === this.selectedTableId) || null
    }
  },
  methods: {
    onToolChange(tool) {
      this.currentTool = tool
      if (tool !== 'cursor') {
        this.selectedTableId = null
      }
    },
    onTableSelected(tableId) {
      this.selectedTableId = tableId
      if (tableId) {
        this.currentTool = 'cursor'
      }
    },
    onTableUpdated(updatedTable) {
      const index = this.tables.findIndex(t => t.id === updatedTable.id)
      if (index !== -1) {
        this.tables.splice(index, 1, updatedTable)
      }
    },
    onCreateTable(tableData) {
      const newTable = {
        id: this.nextTableId++,
        x: tableData.x,
        y: tableData.y,
        rows: tableData.rows || 2,
        cols: tableData.cols || 2,
        cellWidth: 150,
        cellHeight: 50,
        cells: this.createEmptyCells(tableData.rows || 2, tableData.cols || 2),
        style: {
          fontSize: 14,
          fontFamily: 'Inter, sans-serif',
          fontStyle: 'normal',
          textDecoration: 'none',
          fill: '#1e1e1e',
          align: 'left'
        }
      }
      this.tables.push(newTable)
      this.selectedTableId = newTable.id
      this.currentTool = 'cursor'
    },
    createEmptyCells(rows, cols) {
      const cells = []
      for (let r = 0; r < rows; r++) {
        const row = []
        for (let c = 0; c < cols; c++) {
          row.push({ text: '' })
        }
        cells.push(row)
      }
      return cells
    },
    updateTableStyle(styleUpdate) {
      if (this.selectedTable) {
        const updated = { ...this.selectedTable }
        updated.style = { ...updated.style, ...styleUpdate }
        this.onTableUpdated(updated)
      }
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  overflow: hidden;
}

.app-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f5f5;
}

.canvas-container {
  flex: 1;
  position: relative;
  overflow: hidden;
}
</style>
