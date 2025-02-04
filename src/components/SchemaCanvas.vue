<template>
  <div class="schema-canvas">
    <SvgComponent 
      ref="svgCanvas" 
      width="100%" 
      height="100%" />

    <!-- Таблицы -->
    <div
      v-for="table in tables"
      :key="table.id"
      class="table"
      :style="{ left: table.x + 'px', top: table.y + 'px' }"
      @mousedown="startDrag(table, $event)"
    >
      <div class="table-header">{{ table.name }}</div>
      <div v-for="column in table.columns" :key="column.name" class="table-column">
        {{ column.name }} ({{ column.type }})
      </div>
    </div>

    <!-- Связи -->
    <svg>
      <path
        v-for="(relation, index) in relations"
        :key="index"
        :d="generateRelationPath(relation)"
        stroke="black"
        fill="none"
      />
    </svg>
  </div>
</template>

<script>
import { SVG as SvgComponent } from 'svg.js';

export default {
  props: ['tables', 'relations'],
  components: { SvgComponent },
  watch:{
  },
  methods: {
    startDrag(table, event) {
      const startX = event.clientX;
      const startY = event.clientY;
      const initialX = table.x;
      const initialY = table.y;

      const moveHandler = (e) => {
        const dx = e.clientX - startX;
        const dy = e.clientY - startY;
        table.x = initialX + dx;
        table.y = initialY + dy;
      };

      const upHandler = () => {
        window.removeEventListener('mousemove', moveHandler);
        window.removeEventListener('mouseup', upHandler);
      };

      window.addEventListener('mousemove', moveHandler);
      window.addEventListener('mouseup', upHandler);
    },
    generateRelationPath(relation) {
      // Логика расчета пути связи (упрощенно)
      const fromTable = this.tables.find(t => t.id === relation.from);
      const toTable = this.tables.find(t => t.id === relation.to);
      return `M ${fromTable.x + 100} ${fromTable.y + 50} L ${toTable.x} ${toTable.y + 50}`;
    }
  }
};
</script>

<style scoped>
.schema-canvas {
  position: relative;
  background: #f0f0f0;
  overflow: auto;
}

.table {
  position: absolute;
  background: white;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 200px;
  cursor: move;
}

.table-header {
  background: #4CAF50;
  color: white;
  padding: 8px;
  font-weight: bold;
}

.table-column {
  padding: 4px 8px;
  border-bottom: 1px solid #eee;
}
</style>