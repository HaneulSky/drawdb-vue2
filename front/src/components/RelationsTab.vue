<template>
  <div class="relations-tab-container">
    <div class="relations-tab-head">
      <!-- TODO здесь селект -->
      <input
        v-model="search"
        type="search"
        placeholder="Поиск"
        class="relations-search-input"
      >
      <button
        class="add-relation-btn"
        @click="addNewRelation">
        Добавить связь
      </button>
    </div>
    <div
      v-if="relations.length"
      class="relations-tab-relations"
    >
      <details 
        v-for="relation in localValue"
        :key="relation.id" 
        :open="editRelation.id === relation.id"
      >
        <summary>{{`${relation.from?.name || '(пусто)'} (${relation.type}) ${relation.to?.name || '(пусто)'}` || 'new relation' }}</summary>
        <TabsRelationBlock
          :relation="relation"
          :tables="tables"
          :datatypes="datatypes"
        />
      </details>
    </div>
  </div>
</template>
<script>
import TabsRelationBlock from './TabsRelationBlock.vue';

export default {
  name: 'RelationsTab',
  components: { TabsRelationBlock },
  props: {
    relations: {
      type: Array,
      default: () => ([])
    },
    tables: {
      type: Array,
      default: () => ([])
    },
    editRelation: {
      type: Object,
      default: () => ({})
    },
    datatypes: {
      type: Array,
      default: () => ([])
    }, 
  },
  data(){
    return {
      search: null,
      localValue: []
    }
  },
  watch: {
    relations:{
      immediate: true,
      deep: true,
      handler() {
        this.localValue = this.relations
      }
    } 
  },
  methods: {
    addNewRelation(){
      this.localValue.push({
        id: `new-${this.localValue.length + 1}`,
        type: null,
        from: null,
        to: null
      })
    }
  }
}
</script>
<style scoped>
.relations-tab-head {
    display: flex;
    flex-wrap: nowrap;
    gap: 10px;
    width: 100%;
    margin-bottom: 25px;
    position: sticky;
    top: 0;
}

.add-relation-btn {
    cursor: pointer;
    border: 1px solid #3f496a;
    min-height: 24px;
}

.relations-search-input {
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
    margin-bottom: 0;
}
</style>