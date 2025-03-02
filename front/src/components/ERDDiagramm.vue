<template>
  <div class="erd-container">
    <Toolbar 
      :tables="tables"
      :relations="relations" 
      :edit-table-id="editableTableId"
      :datatypes="datatypes"
      @edit-table="$emit('edit-table', $event)"
      @edit-relation="$emit('edit-ralation', $event)"
      @add-new-table="$emit('add-new-table')"
    />
    <SvgCanvas 
      :tables="tables"
      :relations="relations"
      @drag-table="$emit('drag-table', $event)"
      @edit-table="editableTableId=$event" 
      @update:table-position="$emit('update:table-position', $event)"
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
  methods:{

  }
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