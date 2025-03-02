<template>
  <div
    class="table-block"
    @mousedown="$emit('mousedown', $event)"
  >
    <div class="table-block-header">
      {{ tableData.name }}
      <button 
        class="material-icons"
        @click="$emit('edit:table', tableData.id)"
      >
        edit
      </button>
    </div>
    <div
      v-if="tableData[fieldsName]"
      class="table-block-data"
    >
      <div
        v-for="field in tableData[fieldsName]"
        :key="field.id"
        class="table-block-data-field"
      >
        <button
          class="field-indicator"
          @click="$emit('start-relation')"/>
        <span>{{ field.name }}</span>
        <span>{{ field.type }}</span>
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
    },
    fieldsName: {
      type: String,
      default: 'fields'
    },
  },
}
</script>
<style scoped>
.table-block {
    width: 100%;
    height: auto;
    border: 1px solid #494949;
    border-radius: 5px;
    box-sizing: border-box;
    background: white;
    cursor: move;
    pointer-events: all;
}

.table-block:hover {
    border: 1px dashed #3f496a;
}

.table-block-header {
    background: #dadada;
    height: 40px;
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
    height: 40px;
}

.table-block-data-field:last-child {
    border: none;
}
</style>