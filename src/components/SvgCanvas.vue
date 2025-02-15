<template>
    <div 
        class="canvas-container"
        ref="svgCanvasBlock"
        droppable
        @mousemove="handleDrag"
        @mouseup="stopDrag"
     >
        <svg
            v-if="canvasSize"
            ref="svgCanvas"
            :view-box="viewBox"
        >
           <!-- Фоновый узор из точек -->
            <pattern 
                id="bg-pattern" 
                patternUnits="userSpaceOnUse" 
                width="15" 
                height="15"
            >
                <circle cx="15" cy="15" r="1" fill="#a0a0a0" />
            </pattern>
            
            <!-- Заполняем весь холст паттерном -->
            <rect 
                x="0" 
                y="0" 
                width="100%" 
                height="100%" 
                fill="url(#bg-pattern)" 
                pointer-events="none"
            />
            <foreignObject
                v-for="table in tables"
                :key="table.id"
                :x="table.x"
                :y="table.y"
                width="170"
                :height="getBlockHeight(table)"
            >
                <CanvasTableBlock
                    :table-data="table"
                    @mousedown="startDrag(table, $event)"
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
            default: () => ([])
        },
        relations: {
            type: Array,
            default: () => ([])
        }
    },
    data(){
        return {
            canvasSize: null,
            viewBox: null,
            observer: null,
            transform: {
                zoom: 1,
                pan: { x: 0, y: 0 },
            },
            dragData: {
                isDragging: false,
                activeItem: null,
                offsetX: 0,
                offsetY: 0
            }
        }
    },
    computed: {},
    watch: {
        viewBox(val){
            console.log('here')
            if(!val) return;
            const sizes = this.viewBox.split(' ');
            this.canvasSize = { width: sizes[2], height: sizes[3] };
        }
    },
    mounted(){
        this.observer = new ResizeObserver((entries) => {
            for (const entry of entries) {
                this.handleResize(entry);
            }
        });

        if (this.$refs.svgCanvasBlock) {
            this.observer.observe(this.$refs.svgCanvasBlock);
        }
    },
    beforeDestroy(){
        if (this.observer) {
        this.observer.disconnect();
        this.observer = null;
    }
    },
    methods: {
        handleResize(entry) {
            const { width, height } = entry.contentRect;
            this.viewBox = `-300 -300 ${width} ${height}`
            this.$emit('resize', { width, height });
        },
        getBlockHeight(table){
            const tableHeadHeight = 36; // с учетом бордеров
            const fieldHeight = 30;

            return table.fields.length ? tableHeadHeight + fieldHeight * table.fields.length : tableHeadHeight;
        },
        startDrag(table, event){
            
            const svgPoint = this.getSVGCoordinates(event)
      
            this.dragData = {
                isDragging: true,
                activeItem: table,
                offsetX: svgPoint.x - table.x,
                offsetY: svgPoint.y - table.y
            }
            
            document.addEventListener('mouseup', this.stopDrag)
        },
        // Обработка перемещения
        handleDrag(event) {
            if (!this.dragData.isDragging) return
            
            const svgPoint = this.getSVGCoordinates(event)
            this.dragData.activeItem.x = svgPoint.x - this.dragData.offsetX
            this.dragData.activeItem.y = svgPoint.y - this.dragData.offsetY
        },

        // Остановка перетаскивания
        stopDrag() {
            this.$emit('drag-table', this.dragData)
            this.dragData.isDragging = false
            document.removeEventListener('mouseup', this.stopDrag)
        },

        // Преобразование координат мыши в SVG-пространство
        getSVGCoordinates(event) {
            const CTM = this.$refs.svgCanvas.getScreenCTM()
            return {
                x: (event.clientX - CTM.e) / CTM.a,
                y: (event.clientY - CTM.f) / CTM.d
            }
        }
    }
}
</script>
<style scoped>
.canvas-container {
    width: 100%;
    height: 100%;
}

.canvas-container svg {
    width: 100%;
    height: 100%;
}
</style>