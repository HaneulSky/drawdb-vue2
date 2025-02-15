<template>
    <div class="table-block" @mousedown="$emit('mousedown', $event)">
        <div class="table-block-header">
            {{ tableData.name }}
            <button 
                class="material-icons"
                @click="$emit('edit')"
            >
                edit
            </button>
            <button 
                class="material-icons" 
                @click="$emit('more-info')"
            >
                more_horiz
            </button>
        </div>
        <div v-if="tableData.fields" class="table-block-data">
            <div
             v-for="field in tableData.fields"
              :key="field.id"
              class="table-block-data-field"
              >
                <button class="field-indicator" @click="$emit('start-relation')"></button> <span>{{ field.name }}</span><span>{{ field.type }}</span>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: 'CanvasTableBlock',
    props: {
        tableData:{
            type: Object,
            default: () => ({})
        }
    },
    data(){
        return {
            
        }
    }
}
</script>
<style scoped>
.table-block {
    width: 170px;
    height: auto;
    border: 2px solid #494949;
    border-radius: 5px;
    box-sizing: border-box;
    background: white;
    /* z-index: 0; */
    cursor: move;
    pointer-events: all;
}

.table-block:hover {
    border: 2px dashed #3f496a;
    /* z-index: 2; */
}

.table-block-header {
    background: #dadada;
    height: 32px;
}

button {
    width: 17px;
    height: 17px;
    font-size: 15;
}

.field-indicator {
    min-width: 10px;
    max-width: 10px;
    min-height: 10px;
    max-height: 10px;
    border-radius: 100%;
    background-color: crimson;
}

.table-block-data-field {
    border-bottom: 1px solid #494949;
    height: 30px;
}

.table-block-data-field:last-child {
    border: none;
}
</style>