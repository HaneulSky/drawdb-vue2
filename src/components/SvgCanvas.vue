<template>
    <div 
        class="canvas-container"
        ref="svgCanvasBlock"
        @mousemove="handleDrag"
        @mouseup="stopDrag"
        @wheel.prevent="handleWheel"
    >
        <svg
            v-if="viewBox.width"
            ref="svgCanvas"
            :viewBox="viewBoxValue"
            @mousedown="handleSvgMouseDown"
            @mousemove="handleMouseMove"
            @mouseup="endPan"
            @mouseleave="endPan"
        >
            <!-- паттерн с трансформацией -->
            <pattern 
                id="bg-pattern" 
                patternUnits="userSpaceOnUse" 
                :width="15 * scale" 
                :height="15 * scale"
                :patternTransform="`translate(${viewBox.x % 15},${viewBox.y % 15})`"
            >
                <circle cx="7.5" cy="7.5" r="1" fill="#a0a0a0" />
            </pattern>

            <!-- Фоновая заливка -->
            <rect 
                :x="viewBox.x - 1000" 
                :y="viewBox.y - 1000" 
                :width="viewBox.width + 1000" 
                :height="viewBox.height + 1000" 
                fill="url(#bg-pattern)" 
                pointer-events="none"
            />

             <!-- Стрелки между объектами -->
            <defs>
                <marker
                id="arrowhead"
                markerWidth="10"
                markerHeight="7"
                refX="9"
                refY="3.5"
                orient="auto"
                >
                <polygon points="0 0, 10 3.5, 0 7" fill="#666" />
                </marker>
            </defs>

            <g class="connections">
                <path
                v-for="relation in relations"
                :key="relation.id"
                :d="getConnectionPath(relation)"
                stroke="#666"
                stroke-width="1.5"
                fill="none"
                marker-end="url(#arrowhead)"
                :data-source="relation.source"
                :data-target="relation.target"
                />
            </g>

            <!-- Объекты таблиц -->
            <foreignObject
                v-for="table in tables"
                :key="table.id"
                :x="dragData?.activeItem?.id === table.id ? dragData.activeItem.x : table.x"
                :y="dragData?.activeItem?.id === table.id ? dragData.activeItem.y : table.y"
                :width="170*scale"
                :height="getBlockHeight(table)"
                :style="contentStyle"
                @mousedown="startDrag(table, $event)"
            >
                <CanvasTableBlock
                    :table-data="table"
                    @mousedown.native.stop="handleTableMouseDown(table, $event)"
                    @edit="$emit('edit-table', $event)"
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
        tables: Array,
        relations: {
            type: Array,
            default: () => [],
            validator: (rels) => rels.every(r => r.source && r.target)
        }
    },
    data() {
        return {
            scale: 1,
            minScale: 0.5,
            maxScale: 4,
            originalSizes: { width: 0, height: 0 },
            viewBox: { x: -300, y: -300, width: 0, height: 0 },
            panning: {
                isPanning: false,
                startX: 0,
                startY: 0,
                startViewBoxX: 0,
                startViewBoxY: 0
            },
            dragData: {
                isDragging: false,
                activeItem: null,
                offsetX: 0,
                offsetY: 0
            },
            observer: null,
            bgUpdateRequested: false,
        };
    },
    computed: {
        viewBoxValue() {
            return `${this.viewBox.x} ${this.viewBox.y} ${this.viewBox.width} ${this.viewBox.height}`;
        },
        contentStyle() {
            return {
                transform: `scale(${1/this.scale})`,
                'transform-origin': '0 0',
                'pointer-events': this.dragData.isDragging ? 'none' : 'auto'
            };
        }
    },
    mounted() {
        this.initResizeObserver();
    },
    beforeDestroy() {
        this.disconnectResizeObserver();
    },
    methods: {
        handleTableMouseDown(table, event) {
            this.startDrag(table, event);
            event.stopPropagation();
        },
        // Добавляем метод для обновления фона
        updateBackground() {
            if (!this.bgUpdateRequested && this.$refs.svgCanvas) {
                this.bgUpdateRequested = true;
                requestAnimationFrame(() => {
                const bgRect = this.$refs.svgCanvas.querySelector('#background-rect');
                if (bgRect) {
                    bgRect.setAttribute('x', Math.floor(this.viewBox.x / 100) * 100 - 1000);
                    bgRect.setAttribute('y', Math.floor(this.viewBox.y / 100) * 100 - 1000);
                }
                this.bgUpdateRequested = false;
                });
            }
        },
        // Инициализация Resize Observer
        initResizeObserver() {
            this.observer = new ResizeObserver(entries => {
                for (const entry of entries) this.handleResize(entry);
            });
            if (this.$refs.svgCanvasBlock) {
                this.observer.observe(this.$refs.svgCanvasBlock);
            }
        },

        // Обработка изменения размеров
        handleResize(entry) {
            // Добавляем проверку на актуальность размеров
            const { width, height } = entry.contentRect;
            if (width === this.originalSizes.width && height === this.originalSizes.height) return;
            
            // Используем requestAnimationFrame для разрыва цикла
            requestAnimationFrame(() => {
                this.viewBox.width = width;
                this.viewBox.height = height;
                this.originalSizes = { width, height };
                this.$emit('resize', { width, height });
            });
        },

        // Обработчик начала перетаскивания
        startDrag(table, event) {
            if (this.panning.isPanning) return;
            
            const svgPoint = this.getSVGCoordinates(event);
            this.dragData = {
                isDragging: true,
                activeItem: table,
                offsetX: svgPoint.x - table.x,
                offsetY: svgPoint.y - table.y
            };
        },

        // Обработка перемещения мыши
        handleDrag(event) {
            if (this.panning.isPanning || !this.dragData.isDragging) return;
            
            const svgPoint = this.getSVGCoordinates(event);
            const newX = svgPoint.x - this.dragData.offsetX;
            const newY = svgPoint.y - this.dragData.offsetY;

            this.dragData.activeItem.x = newX;
            this.dragData.activeItem.y = newY;
        },

        // Остановка перетаскивания
        stopDrag() {
            if (this.dragData.isDragging) {
                this.$emit('update:table-position', {
                table: this.dragData.activeItem,
            });
                this.dragData.isDragging = false;
            }
        },

        // Панорамирование холста
        handleSvgMouseDown(event) {
            if (event.target.closest('foreignObject')) return;
            
            this.panning = {
                isPanning: true,
                startX: event.clientX,
                startY: event.clientY,
                startViewBoxX: this.viewBox.x,
                startViewBoxY: this.viewBox.y
            };
        },


        // Модифицируем методы панорамирования и зума
        handleMouseMove(event) {
            if (!this.panning.isPanning) return;
            
            const dx = event.clientX - this.panning.startX;
            const dy = event.clientY - this.panning.startY;
            
            this.viewBox.x = this.panning.startViewBoxX - dx / this.scale;
            this.viewBox.y = this.panning.startViewBoxY - dy / this.scale;
            
            this.updateBackground();
        },

        endPan() {
            this.panning.isPanning = false;
        },

        // Преобразование координат мыши
        getSVGCoordinates(event) {
            const pt = this.$refs.svgCanvas.createSVGPoint();
            pt.x = event.clientX;
            pt.y = event.clientY;
            return pt.matrixTransform(
                this.$refs.svgCanvas.getScreenCTM().inverse()
            );
        },

        // Масштабирование
        handleWheel(e) {
            const prevScale = this.scale;
            const delta = e.deltaY > 0 ? 0.9 : 1.1;
            const newScale = Math.min(
                Math.max(this.scale * delta, this.minScale), 
                this.maxScale
            );
            
            const mousePos = this.getSVGCoordinates(e);
            const ratio = 1 - newScale / this.scale;
            
            this.viewBox.x += mousePos.x * ratio;
            this.viewBox.y += mousePos.y * ratio;
            this.viewBox.width = this.originalSizes.width / newScale;
            this.viewBox.height = this.originalSizes.height / newScale;
            this.scale = newScale;

            if (Math.abs(prevScale - this.scale) > 0.001) {
                this.$nextTick(() => {
                    this.observer?.unobserve(this.$refs.svgCanvasBlock);
                    this.observer?.observe(this.$refs.svgCanvasBlock);
                });
            }

            this.updateBackground();
        },

        getBlockHeight(table) {
            const tableHeadHeight = 36;
            const fieldHeight = 30;
            return tableHeadHeight + (table.fields?.length || 0) * fieldHeight;
        },

        disconnectResizeObserver() {
            if (this.observer) {
                this.observer.disconnect();
                this.observer = null;
            }
        },

            // Расчет пути для стрелки
        getConnectionPath(relation) {
            const sourceTable = this.tables.find(t => t.id === relation.source);
            const targetTable = this.tables.find(t => t.id === relation.target);
            if (!sourceTable || !targetTable) return '';

            const start = this.getConnectionPoint(sourceTable, 'right');
            const end = this.getConnectionPoint(targetTable, 'left');
            const curvature = 80 * this.scale;

            return `
                M ${start.x},${start.y}
                C ${start.x + curvature},${start.y}
                ${end.x - curvature},${end.y}
                ${end.x},${end.y}
            `;
        },

        // Расчет точки соединения
        getConnectionPoint(table, side) {
            const width = 170 * this.scale;
            const height = this.getBlockHeight(table);
            
            const centerY = table.y + height / 2;
            
            switch(side) {
                case 'left':
                return { x: table.x, y: centerY };
                case 'right':
                return { x: table.x + width, y: centerY };
                default:
                return { x: table.x + width/2, y: table.y + height };
            }
        },
    }
};
</script>

<style scoped>
.canvas-container {
    width: 100%;
    height: 100%;
    overflow: hidden;
    cursor: grab;
    position: relative;
    contain: strict;
    transform: translateZ(0);
    will-change: transform;
    overflow: hidden;
    background: #fffdfd;
}

.canvas-container:active {
    cursor: grabbing;
}

svg {
    touch-action: none;
    user-select: none;
}

foreignObject {
  will-change: transform;
  transform: translateZ(0);
  cursor: move;
}

pattern circle {
    shape-rendering: crispEdges;
}

.connections path {
  pointer-events: none; /* Или 'stroke' для взаимодействия */
  transition: d 0.3s ease-out;
}

@media (prefers-reduced-motion: reduce) {
  .connections path {
    transition: none;
  }
}
</style>