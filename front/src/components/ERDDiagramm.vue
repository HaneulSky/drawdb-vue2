<template>
  <div class="erd-container">
    <Toolbar 
      :tables="tables"
      :relations="relations" 
      :edit-table-id="editableTableId"
      :datatypes="datatypes"
      @edit:table="$emit('edit-table', $event)"
      @edit:relation="$emit('edit-relation', $event)"
      @delete:relation="$emit('delete:relation', $event)"
      @create:table="$emit('create:table')"
      @update:table="$emit('update:table', $event)"
      @delete:table="$emit('delete:table', $event)"
      @create:field="$emit('create:field',$event)"
    />
    <SvgCanvas 
      :tables="tables"
      :relations="relations"
      @drag-table="$emit('drag-table', $event)"
      @edit:table="editableTableId=$event"
      @update:table="$emit('update:table', $event)"
    />
  </div>
</template>
<script>
import Toolbar from './ErdToolbar.vue';
import SvgCanvas from './SvgCanvas.vue';

export default {
  name: 'ERDDiagramm',
  components: { Toolbar, SvgCanvas },
  props: {
    tables: {
      type: Array,
      default: () => ([])
    },
    relations: {
      type: Array,
      default: () => ([])
    },
    datatypes: {
      type: Array,
      default: () => ([])
    },
  },
  data(){
    return {
      editableTableId: null
    }
  },
}
</script>
<style scoped>
.erd-container {
  display: grid;
  grid-template-columns: 25% 75%;
  width: 100%;
  overflow: hidden;
}
</style>