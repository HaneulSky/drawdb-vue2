<template>
    <div class="toolbar">
      <button @click="addTable">Добавить таблицу</button>
      <button @click="addRelation">Добавить связь</button>
      <button @click="exportSchema">Экспорт</button>
    </div>
</template>
  
  <script>
  export default {
    props: {
      tables: {
        type: [Array, null],
        default: null
      }
    },
    methods: {
      addTable() {
        this.$emit('add-table', {
          id: this.tables?.length + 1 || 1,
          name: 'New Table',
          x: 50,
          y: 50,
          columns: [{ name: 'id', type: 'INT' }]
        });
      },
      addRelation() {
        // Логика выбора таблиц для связи
        this.$emit('add-relation', { from: 1, to: 2 });
      },
      exportSchema() {
        this.$emit('export');
      }
    }
  };
  </script>
  
  <style scoped>
  .toolbar {
    padding: 10px;
    background: #e0e0e0;
    margin-bottom: 20px;
    display: flex;
    flex-direction: column;
    height: 100%;
    gap: 10px;
    overflow: auto;
  }
  
  button {
    margin-right: 10px;
    padding: 8px 12px;
    height: 45px;
  }
  </style>