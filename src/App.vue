<template>
  <div>
    <ERDDiagramm 
      :tables="tables" 
      @add-new-table="addTable"
      @update:table-position="updateTables"
    />
  </div>
</template>

<script>
import ERDDiagramm from './components/Main.vue';

export default {
  components: { ERDDiagramm },
  data() {
    return {
      tables: [
        {
          id: 1,
          name: 'New Table 1',
          comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
          x: 10,
          y: 25,
          relations: [],
          fields: [
            {
              id: 1,
              name: 'id',
              nullable: false,
              unique: true, 
              increment: true,
              comment: null
            }
          ]
        }
      ],
      relations: []
    };
  },
  methods: {
    updateTables({ table }){
      const tableIndex = this.tables.findIndex(t => t.id === table.id);
      this.tables[tableIndex] = {...this.tables[tableIndex], ...table}
    },
    addTable() {
      this.tables.push({
          id: (this.tables.length || 0) + 1,
          name: `New Table ${(this.tables.length || 0) + 1}`,
          comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
          x: 10,
          y: 25,
          relations: [],
          fields: [
            {
              id: 1,
              name: 'id',
              nullable: false,
              unique: true, 
              increment: true,
              comment: null
            }
          ]
        });
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
<style scoped>
.erd-container {
  display: grid;
  min-height: 100vh;
  overflow: hidden;
}
</style>