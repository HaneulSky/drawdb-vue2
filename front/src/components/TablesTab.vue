<template>
  <div class="tables-tab-container">
    <div class="tables-tab-head">
      <input
        v-model="search"
        type="search"
        placeholder="Поиск"
        class="tables-search-input"
      >
      <button
        class="add-table-btn"
        @click="$emit('create:table')"
      >
        Добавить таблицу
      </button>
    </div>
    <div
      v-if="tables.length"
      class="tables-tab-tables"
    >
      <details 
        v-for="table in tables"  
        :key="table.id"
        :open="editTableId === table.id"
      >
        <summary>{{ table.name || 'таблица' }}</summary>
        <TabsTableBlock
          :table="table"
          @create:field="$emit('create:field',$event)"
          @update:table="$emit('update:table',$event)"
          @delete:table="$emit('delete:table',$event)"
        />
      </details>
    </div>
  </div>
</template>
<script>
import TabsTableBlock from './TabsTableBlock.vue';

export default {
  name: 'TablesTab',
  components: { TabsTableBlock },
  props: {
    tables: {
      type: Array,
      default: () => ([])
    },
    editTableId: {
      type: [String, Number],
      default: 1
    },
    datatypes: {
      type: Array,
      default: () => ([])
    },
  },
  data(){
    return {
      search: null,
    }
  }
}
</script>
<style scoped>
.tables-tab-container {
  max-height: 100%;
}

.tables-tab-head {
  display: flex;
  flex-wrap: nowrap;
  gap: 10px;
  width: 100%;
  margin-bottom: 25px;
  position: sticky;
  top: 0;
  background: white;
}

.tables-tab-tables {
  height: 100%;
}

.add-table-btn {
  cursor: pointer;
  border: 1px solid #3f496a;
  min-height: 24px;
}

.tables-search-input {
  width: 100%;
}

summary {
  cursor: pointer;
}

details:open summary {
  border-bottom: 1px dashed #3f496a;
  padding-bottom: 5px;
}

details {
  border: 1px solid #494949;
  border-radius: 5px;
  padding: 5px;
  margin-bottom: 10px;
}

details:last-child {
  margin-bottom: none;
}
</style>