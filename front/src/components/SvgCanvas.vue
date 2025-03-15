<template>
  <div
    ref="svgCanvasBlock"
    class="canvas-container"
    @mousemove="handleDrag"
    @mouseup="stopDrag"
    @wheel.prevent="handleWheel"
  >
    <div class="scale-value">
      <button
        class="material-icons"
        :disabled="scale===maxScale"
        @click="onScaleButton('plus')"
      >
        +
      </button>
      {{ `${Math.round(100 * scale)}%` }}
      <button
        class="material-icons"
        :disabled="scale===minScale"
        @click="onScaleButton('minus')"
      >
        -
      </button>
    </div>
    <svg
      v-if="viewBox.width"
      ref="svgCanvas"
      :viewBox="viewBoxValue"
      @mousedown="handleSvgMouseDown"
      @mousemove="handleMouseMove"
      @mouseup="endPan"
      @mouseleave="endPan"
    >
      <!-- Паттерн с трансформацией с точками -->
      <pattern
        id="bg-pattern"
        patternUnits="userSpaceOnUse"
        :width="15 * scale"
        :height="15 * scale"
        :patternTransform="`translate(${viewBox.x % 15},${viewBox.y % 15})`"
      >
        <circle
          cx="7.5"
          cy="7.5"
          r="1"
          fill="#dadada"
        />
      </pattern>

      <defs>
        <!-- Маркер для "Многие" (M) справа -->
        <marker
          id="arrowhead-many"
          markerWidth="16"
          markerHeight="16"
          refX="15"
          refY="8.5"
          orient="auto"
          markerUnits="strokeWidth"
        >
          <path
            d="M1 17L17 8.5M17 8.5L1 1M17 8.5H1"
            stroke="#888A90"
            stroke-linejoin="round"
            stroke-width="1.5"
          />
        </marker>
        <!-- Маркер для "Многие" (M) слева -->
        <marker
          id="arrowhead-many-left"
          markerWidth="16"
          markerHeight="16"
          refX="15"
          refY="8.5"
          orient="auto-start-reverse"
          markerUnits="strokeWidth"
        >
          <path
            d="M1 17L17 8.5M17 8.5L1 1M17 8.5H1"
            stroke="#888A90"
            stroke-linejoin="round"
            stroke-width="1.5"
            transform="rotate(180 9 8.5)"
          />
        </marker>
        <!-- Маркер для "Один" (O) -->
        <marker
          id="crossbar-one"
          markerWidth="16"
          markerHeight="16"
          refX="15"
          refY="8"
          orient="auto"
          markerUnits="userSpaceOnUse"
        >
          <path
            d="M16 8H0M8 16L8 0"
            stroke="#888A90"
            stroke-linejoin="round"
            :stroke-width="1.5"
          />
        </marker>
      </defs>


      <!-- Фоновая заливка -->
      <rect
        :x="viewBox.x - 1000"
        :y="viewBox.y - 1000"
        :width="viewBox.width + 1000"
        :height="viewBox.height + 1000"
        fill="url(#bg-pattern)"
        pointer-events="none"
      />
      <g class="connections">
        <path
          v-for="relation in relationsData"
          :key="relation.id"
          :d="getConnectionPath(relation)"
          stroke="#888A90"
          :stroke-width="1.5"
          fill="none"
          :marker-start="getMarkerStart(relation)"
          :marker-end="getMarkerEnd(relation)"
          @dblclick="$emit('edit:relation', relation.id)"
        />
      </g>

      <!-- Объекты таблиц -->
      <foreignObject
        v-for="table in tables"
        :key="table.id"
        :x="dragData?.activeItem?.id === table.id ? dragData.activeItem.xAxis || 1 : (table.xAxis || 1)"
        :y="dragData?.activeItem?.id === table.id ? dragData.activeItem.yAxis || 1 : (table.yAxis || 1)"
        :width="tableWidth"
        :height="getBlockHeight(table)"
        @mousedown="startDrag(table, $event)"
      >
        <CanvasTableBlock
          :table-data="table"
          @mousedown.native.stop="handleTableMouseDown(table, $event)"
          @edit:table="$emit('edit:table', $event)"
          @edit:field="$emit('edit:field', $event)"
        />
      </foreignObject>
    </svg>
  </div>
</template>
<script>
import CanvasTableBlock from './CanvasTableBlock.vue';

export default {
  name: 'SvgCanvas',
  components: { CanvasTableBlock },
  props: {
    tables: {
      type: Array,
      default: () => [],
    },
    relations: {
      type: Array,
      default: () => [],
    },
    localStorageName: {
      type: String,
      default: 'erd-diagram'
    },
    fieldsName: {
      type: String,
      default: 'fields'
    },
  },
  data() {
    return {
      scale: 1,
      minScale: 0.5,
      maxScale: 4,
      originalSizes: { width: 0, height: 0 },
      viewBox: {
        x: -300, y: -300, width: 0, height: 0,
      },
      panning: {
        isPanning: false,
        startX: 0,
        startY: 0,
        startViewBoxX: 0,
        startViewBoxY: 0,
      },
      dragData: {
        isDragging: false,
        activeItem: null,
        offsetX: 0,
        offsetY: 0,
        type: null,
      },
      observer: null,
      bgUpdateRequested: false,
      tableWidth: 200,
      relationsData: [],
    };
  },
  computed: {
    viewBoxValue() {
      return `${this.viewBox.x} ${this.viewBox.y} ${this.viewBox.width} ${this.viewBox.height}`;
    },
  },
  watch: {
    relations: {
      immediate: true,
      deep: true,
      handler() {
        this.relationsData = JSON.parse(JSON.stringify(this.relations));
      },
    },
  },
  mounted() {
    const scale = localStorage.getItem(`${this.localStorageName}-erd-scale`);
    const viewBox = localStorage.getItem(`${this.localStorageName}-erd-viewBox`);
    if (viewBox) this.viewBox = JSON.parse(viewBox);
    if (typeof +scale === 'number' && !Number.isNaN(+scale)) this.scale = Math.min(Math.max(+scale, this.minScale), this.maxScale);
    this.initResizeObserver();
    this.handleResize({ contentRect: this.$refs.svgCanvasBlock.getBoundingClientRect() });
  },
  beforeDestroy() {
    this.disconnectResizeObserver();
  },
  methods: {
    onScaleButton(action) {
      const prevScale = this.scale;
      const roundedScale = Math.round(prevScale / 0.25) * 0.25;
      let newScale = action === 'plus' ? roundedScale + 0.25 : roundedScale - 0.25;
      newScale = Math.min(Math.max(newScale, this.minScale), this.maxScale);
      const centerX = this.viewBox.x + this.viewBox.width / 2;
      const centerY = this.viewBox.y + this.viewBox.height / 2;
      const ratio = 1 - newScale / prevScale;
      this.viewBox.x += centerX * ratio;
      this.viewBox.y += centerY * ratio;
      this.viewBox.width = this.originalSizes.width / newScale;
      this.viewBox.height = this.originalSizes.height / newScale;
      this.scale = newScale;
      localStorage.setItem(`${this.localStorageName}-erd-scale`, this.scale);
      localStorage.setItem(`${this.localStorageName}-erd-viewBox`, JSON.stringify(this.viewBox));
      this.updateBackground();
      this.$nextTick(() => {
        this.relationsData = [...this.relationsData];
      });
    },
    handleTableMouseDown(table, event) {
      this.startDrag(table, event);
      event.stopPropagation();
    },
    updateBackground() {
      if (!this.bgUpdateRequested && this.$refs.svgCanvas) {
        this.bgUpdateRequested = true;
        requestAnimationFrame(() => {
          const bgRect = this.$refs.svgCanvas.querySelector('#background-rect');
          if (bgRect) {
            bgRect.x.baseVal.value = Math.floor(this.viewBox.x / 100) * 100 - 1000;
            bgRect.y.baseVal.value = Math.floor(this.viewBox.y / 100) * 100 - 1000;
          }
          this.bgUpdateRequested = false;
        });
      }
    },
    initResizeObserver() {
      this.observer = new ResizeObserver(entries => {
        for (const entry of entries) this.handleResize(entry);
      });
      if (this.$refs.svgCanvasBlock) this.observer.observe(this.$refs.svgCanvasBlock);
    },
    handleResize(entry) {
      const { width, height } = entry.contentRect;
      if (width === this.originalSizes.width && height === this.originalSizes.height) return;
      requestAnimationFrame(() => {
        this.originalSizes = { width, height };
        this.viewBox.width = width / this.scale;
        this.viewBox.height = height / this.scale;
        this.$emit('resize', { width, height });
      });
      localStorage.setItem(`${this.localStorageName}-erd-viewBox`, JSON.stringify(this.viewBox));
    },
    startDrag(table, event) {
      if (this.panning.isPanning) return;
      const svgPoint = this.getSVGCoordinates(event);
      this.dragData = {
        isDragging: true,
        activeItem: table,
        offsetX: svgPoint.x - (table.xAxis || 0),
        offsetY: svgPoint.y - (table.yAxis || 0),
        type: 'table',
        relation: null,
      };
    },
    handleDrag(event) {
      // Заменяем на эту упрощенную версию
      // Если перетаскивание не активно, ничего не делаем
      if (!this.dragData.isDragging) return;
      const svgPoint = this.getSVGCoordinates(event);
      // Обрабатываем только перетаскивание таблиц
      if (this.dragData.type === 'table') {
        const newX = svgPoint.x - this.dragData.offsetX;
        const newY = svgPoint.y - this.dragData.offsetY;
        this.dragData.activeItem.xAxis = newX;
        this.dragData.activeItem.yAxis = newY;
      }
    },
    stopDrag() {
      // Заменяем на эту упрощенную версию
      // Если перетаскивание не активно, ничего не делаем
      if (!this.dragData.isDragging) return;
      // Обновляем только таблицы
      if (this.dragData.type === 'table') {
        this.$emit('update:table', {
          ...this.dragData.activeItem,
        });
      }
      // Сбрасываем данные перетаскивания
      this.dragData = {
        isDragging: false,
        activeItem: null,
        offsetX: 0,
        offsetY: 0,
        type: null,
      };
    },
    handleSvgMouseDown(event) {
      if (event.target.closest('foreignObject') || event.target.classList.contains('connection-handle')) return;
      this.panning = {
        isPanning: true,
        startX: event.clientX,
        startY: event.clientY,
        startViewBoxX: this.viewBox.x,
        startViewBoxY: this.viewBox.y,
      };
    },
    handleMouseMove(event) {
      if (!this.panning.isPanning) return;
      const dx = event.clientX - this.panning.startX;
      const dy = event.clientY - this.panning.startY;
      this.viewBox.x = this.panning.startViewBoxX - dx / this.scale;
      this.viewBox.y = this.panning.startViewBoxY - dy / this.scale;
      this.updateBackground();
      localStorage.setItem(`${this.localStorageName}-erd-viewBox`, JSON.stringify(this.viewBox));
    },
    endPan() {
      this.panning.isPanning = false;
    },
    getSVGCoordinates(event) {
      const pt = this.$refs.svgCanvas.createSVGPoint();
      pt.x = event.clientX;
      pt.y = event.clientY;
      return pt.matrixTransform(this.$refs.svgCanvas.getScreenCTM().inverse());
    },
    handleWheel(e) {
      if (!e.ctrlKey && !e.metaKey) {
        if (e.shiftKey) this.viewBox.x += e.deltaY > 0 ? 10 : -10;
        else this.viewBox.y += e.deltaY > 0 ? 20 : -20;
        localStorage.setItem(`${this.localStorageName}-erd-viewBox`, JSON.stringify(this.viewBox));
        return;
      }
      e.preventDefault();
      const prevScale = this.scale;
      const delta = e.deltaY > 0 ? 0.9 : 1.1;
      const newScale = Math.min(Math.max(prevScale * delta, this.minScale), this.maxScale);
      const mousePos = this.getSVGCoordinates(e);
      const scaleFactor = prevScale / newScale;
      this.viewBox.x = mousePos.x - (mousePos.x - this.viewBox.x) * scaleFactor;
      this.viewBox.y = mousePos.y - (mousePos.y - this.viewBox.y) * scaleFactor;
      this.viewBox.width = this.originalSizes.width / newScale;
      this.viewBox.height = this.originalSizes.height / newScale;
      this.scale = newScale;
      localStorage.setItem(`${this.localStorageName}-erd-scale`, this.scale);
      localStorage.setItem(`${this.localStorageName}-erd-viewBox`, JSON.stringify(this.viewBox));
      this.updateBackground();
      this.$nextTick(() => {
        this.relationsData = [...this.relationsData];
      });
    },
    getBlockHeight(table) {
      const tableHeadHeight = 42; // с учетом бордера
      const fieldHeight = 42; // с учетом бордера
      return tableHeadHeight + (table[this.fieldsName]?.length || 0) * fieldHeight;
    },
    disconnectResizeObserver() {
      if (this.observer) {
        this.observer.disconnect();
        this.observer = null;
      }
    },
    getMarkerEnd(relation) {
      const sourceTable = this.tables.find(t => t.id === relation.source.id);
      const targetTable = this.tables.find(t => t.id === relation.target.id);
      if (!sourceTable || !targetTable) return '';
      const visualTableWidth = this.tableWidth;
      const sourceVisualCenter = sourceTable.xAxis + visualTableWidth / 2;
      const targetVisualCenter = targetTable.xAxis + visualTableWidth / 2;
      const isSourceLeft = sourceVisualCenter < targetVisualCenter;
      const isSourceAbove = sourceTable.yAxis > targetTable.yAxis;
      const isSameXAxis = Math.abs(sourceTable.xAxis - targetTable.xAxis) <= this.tableWidth;
      const relationType = relation.datatype.name;
      if (relationType === 'MtM') return isSourceLeft ? 'url(#arrowhead-many)' : 'url(#arrowhead-many-left)';
      if (relationType === 'MtO') return (isSourceAbove && !isSameXAxis) || !isSourceAbove ? 'url(#crossbar-one)' : 'url(#arrowhead-many-left)';
      if (relationType === 'OtM') return (isSourceAbove && !isSameXAxis) || !isSourceAbove ? 'url(#arrowhead-many-left)' : 'url(#crossbar-one)';
      return 'url(#crossbar-one)';
    },
    getMarkerStart(relation) {
      const sourceTable = this.tables.find(t => t.id === relation.source.id);
      const targetTable = this.tables.find(t => t.id === relation.target.id);
      if (!sourceTable || !targetTable) return '';
      const isSourceAbove = sourceTable.yAxis > targetTable.yAxis;
      const isSameXAxis = Math.abs(sourceTable.xAxis - targetTable.xAxis) <= this.tableWidth;

      const relationType = relation.datatype.name;
      if (relationType === 'MtM') return 'url(#arrowhead-many-left)';
      if (relationType === 'MtO') return isSourceAbove && isSameXAxis ? 'url(#crossbar-one)' : 'url(#arrowhead-many-left)';
      if (relationType === 'OtM') return isSourceAbove && isSameXAxis ? 'url(#arrowhead-many-left)' : 'url(#crossbar-one)';
      return 'url(#crossbar-one)';
    },
    getConnectionPoint(table, side, relation, isSource = false) {
      const visualTableWidth = this.tableWidth;
      const visualHeaderHeight = 40;
      const fieldHeight = 40;
      const baseX = table.xAxis;
      const baseY = table.yAxis;
      let yPosition;
      const fieldName = isSource ? relation.sourceField.name : relation.targetField.name;
      const field = table.fields.find(f => f.name === fieldName);
      if (field) {
        const fieldIndex = table.fields.findIndex(f => f.name === fieldName);
        yPosition = baseY + visualHeaderHeight + fieldIndex * fieldHeight + fieldHeight / 2;
      } else {
        yPosition = baseY + visualHeaderHeight / 2;
      }
      const offset = 2;
      switch (side) {
      case 'left':
        return { x: baseX - offset, y: yPosition };
      case 'right':
        return { x: baseX + visualTableWidth + offset, y: yPosition };
      default:
        return { x: baseX + visualTableWidth / 2, y: yPosition };
      }
    },
    getConnectionPath(relation) {
      const sourceTable = this.tables.find(t => t.id === relation.source.id);
      const targetTable = this.tables.find(t => t.id === relation.target.id);
      if (!sourceTable || !targetTable) return '';
      const visualTableWidth = this.tableWidth;
      const isSameXAxis = Math.abs(sourceTable.xAxis - targetTable.xAxis) <= visualTableWidth;
      const isSourceAbove = sourceTable.yAxis < targetTable.yAxis;
      const sourceVisualCenterX = sourceTable.xAxis + visualTableWidth / 2;
      const targetVisualCenterX = targetTable.xAxis + visualTableWidth / 2;
      const isSourceLeft = sourceVisualCenterX < targetVisualCenterX;
      const baseRadius = 5;

      if (isSameXAxis) {
        const side = isSourceAbove ? 'right' : 'right';
        const offset = 35;
        const topTable = isSourceAbove ? sourceTable : targetTable;
        const bottomTable = isSourceAbove ? targetTable : sourceTable;
        const topPoint = this.getConnectionPoint(topTable, side, relation, topTable.id === relation.source.id);
        const bottomPoint = this.getConnectionPoint(bottomTable, side, relation, bottomTable.id === relation.source.id);
        const midX = Math.max(topTable.xAxis + visualTableWidth, bottomTable.xAxis + visualTableWidth) + offset;
        if (relation.datatype.name === 'OtO' ||  (relation.datatype.name === 'MtO' && !isSourceAbove) || (relation.datatype.name === 'OtM' && isSourceAbove)) topPoint.x += 15;
        const yDistance = Math.abs(bottomTable.yAxis - topPoint.y);
        const radius = Math.min(baseRadius, yDistance / 2);
        const path = `
          M ${topPoint.x},${topPoint.y}
          L ${midX - radius},${topPoint.y}
          Q ${midX},${topPoint.y} ${midX},${topPoint.y + (bottomPoint.y > topPoint.y ? radius : -radius)}
          L ${midX},${bottomPoint.y - (bottomPoint.y > topPoint.y ? radius : -radius)}
          Q ${midX},${bottomPoint.y} ${bottomPoint.x},${bottomPoint.y}`
        ;
        return path.trim();
      }

      const sourceSide = isSourceLeft ? 'right' : 'left';
      const targetSide = isSourceLeft ? 'left' : 'right';
      const start = this.getConnectionPoint(sourceTable, sourceSide, relation, true);
      const end = this.getConnectionPoint(targetTable, targetSide, relation, false);
      const midX = (start.x + end.x) / 2;

      if ((relation.datatype.name === 'OtO' || relation.datatype.name === 'OtM') && targetSide === 'left') start.x += 15;
      if ((relation.datatype.name === 'OtO' || relation.datatype.name === 'OtM') && targetSide === 'right') start.x -= 15;
      if (start.y === end.y) {
        return `
          M ${start.x},${start.y}
          L ${end.x},${end.y}`
        ;
      }

      const yDistance = Math.abs(end.y - start.y);
      const radius = Math.min(baseRadius, yDistance / 2);
      const path = isSourceLeft
        ? 
        `M ${start.x},${start.y}
          L ${midX - radius},${start.y}
          Q ${midX},${start.y} ${midX},${start.y + (end.y > start.y ? radius : -radius)}
          L ${midX},${end.y - (end.y > start.y ? radius : -radius)}
          Q ${midX},${end.y} ${midX + (end.x > midX ? radius : -radius)},${end.y}
          L ${end.x},${end.y}`
        
        : 
        ` M ${start.x},${start.y}
          L ${midX + radius},${start.y}
          Q ${midX},${start.y} ${midX},${start.y + (end.y > start.y ? radius : -radius)}
          L ${midX},${end.y - (end.y > start.y ? radius : -radius)}
          Q ${midX},${end.y} ${midX + (end.x > midX ? radius : -radius)},${end.y}
          L ${end.x},${end.y}`
        ;
      return path.trim();
    },
    getTable(id) {
      return this.tables.find(t => t.id === id);
    },
    getSide(relation, isSource) {
      const sourceTable = this.tables.find(t => t.id === relation.source.id);
      const targetTable = this.tables.find(t => t.id === relation.target.id);
      if (!sourceTable || !targetTable) return 'right';
      const visualTableWidth = this.tableWidth;
      const sourceVisualCenterX = sourceTable.xAxis + visualTableWidth / 2;
      const targetVisualCenterX = targetTable.xAxis + visualTableWidth / 2;
      const isSourceLeft = sourceVisualCenterX < targetVisualCenterX;
      const isSourceAbove = sourceTable.yAxis < targetTable.yAxis;
      const isSameXAxis = Math.abs(sourceTable.xAxis - targetTable.xAxis) <= visualTableWidth;
      if (isSameXAxis) {
        return isSourceAbove && relation.datatype.name === 'OtO' ? 'left' : 'right';
      }
      return isSource ? (isSourceLeft ? 'right' : 'left') : (isSourceLeft ? 'left' : 'right');
    },
  },
};
</script>

<style scoped>
.canvas-container {
  width: 100%;
  height: 100%;
  cursor: grab;
  position: relative;
  contain: strict;
  transform: translateZ(0);
  will-change: transform;
  background: #fffdfd;
  z-index: 2;
  border: 1px solid #dadada;
  scroll-behavior: smooth;
}

.canvas-container:active {
  cursor: grabbing;
}

.scale-value {
  position: absolute;
  display: flex;
  flex-wrap: nowrap;
  align-items: center;
  justify-content: space-between;
  top: 0;
  right: 2px;
  min-width: 100px;
  max-width: 100px;
  padding: 0 2px;
  height: 20px;
  background: #dadada;
  z-index: 3;
  cursor: pointer;
  font-size: 14px;
}

svg {
  display: block;
  height: 100%;
  width: 100%;
  touch-action: none;
  user-select: none;
  overflow: auto;
}

foreignObject {
  transform-origin: 0 0;
  pointer-events: auto;
  will-change: transform;
  cursor: move;
}

pattern circle {
  shape-rendering: crispEdges;
}

.connections path {
  cursor: pointer;
  pointer-events: stroke;
  transition: d 0.3s ease-out;
}

.connections path:hover {
  stroke: blue;
  box-shadow: 0 0 1px 1px #dadada;
}

.connection-handle {
  cursor: pointer;
}

.connection-handle:hover {
  fill: #888A90;
  opacity: 0.5;
}

button {
  width: 17px;
  height: 17px;
  font-size: 15px;
  padding: 0;
  background: none;
  border: none;
  z-index: 1;
  cursor: pointer;
}

@media (prefers-reduced-motion: reduce) {
  .connections path {
    transition: none;
  }
}
</style>