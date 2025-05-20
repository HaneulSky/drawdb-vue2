<template>
  <div class="erd-toolbar">
    <div class="tabs">
      <button 
        class="tab"
        :class="{'current': currentTab === 0}"
        @click="openTab('tables')"
      >
        Таблицы
      </button>
      <button 
        class="tab"
        :class="{'current': currentTab === 1}"
        @click="openTab('relations')"
      >
        Связи 
      </button>
    </div>
    <div class="tabs-container">
      <TablesTab
        v-show="currentTab === 0"
        :tables="tables"
        :edit-table-id="editTableId"
        :datatypes="notRelationDatatypes"
        @create:table="$emit('create:table')"
        @update:table="$emit('update:table', $event)"
        @delete:table="$emit('delete:table', $event)"
        @create:field="$emit('create:field',$event)"
      />
      <RelationsTab
        v-show="currentTab === 1"
        :relations="relations"
        :tables="tables"
        :datatypes="relationDatatypes"
        @create:relation="$emit('create:relation', $event)"
        @update:relation="$emit('update:relation', $event)"
        @delete:relation="$emit('delete:relation', $event)"
      />
    </div>
  </div>
</template>
<script>
import TablesTab from './TablesTab.vue';
import RelationsTab from './RelationsTab.vue';

export default {
  name: 'ErdToolbar',
  props: {
    tables: {
      type: Array,
      default: () => ([])
    },
    relations: {
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
  components: {TablesTab, RelationsTab},
  data() {
    return {
      currentTab: 0
    }
  },
  computed:{
    relationDatatypes(){
      return this.datatypes.filter((type => ['OtO', 'MtM', 'MtO', 'OtM'].includes(type.name)))
    },
    notRelationDatatypes(){
      return this.datatypes.filter((type => ['OtO', 'MtM', 'MtO', 'OtM'].includes(type.name)))
    }
  },
  methods: {
    openTab(tabName){
      if(tabName === 'tables') this.currentTab = 0;
      if(tabName === 'relations') this.currentTab = 1;
    }
  }
}
</script>
<style scoped>
.erd-toolbar {
  border-right: 1px solid #dadada;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.tabs {
  height: 45px;
  display: grid;
  grid-template-columns: 50% 50%;
}

.tab {
  cursor: pointer;
  border: none;
  background: none;
  border-bottom: 3px solid;
  border-bottom-color: white;
  transition: background .5s ease-in-out, border-bottom-color .5s ease;
}

.tab.current {
  border-bottom-color: rgb(126, 126, 238);
}

.tab:hover {
  background: rgb(233, 250, 255);
  border-bottom-color: #dadada;
}

.tabs-container {
  overflow: auto;
  max-height: calc(100vh - 75px);
  position: relative;
  padding: 10px;
}
</style>