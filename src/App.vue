<template>
  <div>
    <Toolbar
      :tables="tables"
      @add-table="handleAddTable"
      @add-relation="handleAddRelation"
      @export="handleExport"
    />
    <SchemaCanvas :tables="tables" :relations="relations" />
  </div>
</template>

<script>
import SchemaCanvas from './components/SchemaCanvas.vue';
import Toolbar from './components/CanvasToolbar.vue';

export default {
  components: { SchemaCanvas, Toolbar },
  data() {
    return {
      tables: [
        {
          id: 1,
          name: 'New Table 1',
          x: 50,
          y: 50,
          columns: [{ name: 'id', type: 'INT' }]
        },
        {
          id: 2,
          name: 'New Table 2',
          x: 70,
          y: 50,
          columns: [{ name: 'id', type: 'INT' }]
        },
        {
          id: 3,
          name: 'New Table 3',
          x: 90,
          y: 50,
          columns: [{ name: 'id', type: 'INT' }]
        },
      ],
      relations: []
    };
  },
  methods: {
    handleAddTable(newTable) {
      this.tables.push(newTable);
    },
    handleAddRelation(relation) {
      this.relations.push(relation);
    },
    handleExport() {
      const schema = {
        tables: this.tables,
        relations: this.relations
      };
      console.log('Exported Schema:', schema);
      // Можно добавить логику экспорта в JSON/SQL
    }
  }
};
</script>