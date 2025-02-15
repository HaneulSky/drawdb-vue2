<template>
    <div class="relations-tab-container">
        <div class="relations-tab-head">
            <!-- TODO здесь селект -->
            <input v-model="search" type="search" placeholder="Поиск" class="relations-search-input" /> 
            <button class="add-relation-btn" @click="$emit('add-new-relation')">Добавить связь</button>
        </div>
        <div v-if="relations.length" class="relations-tab-relations">
            <details 
                v-for="relation in relations" 
                :key="relation.id"
                :open="editRelation.id === relation.id"
            >
                <summary>{{ relation.name || 'relation' }}</summary>
                <TabsTableBlock :relation="relations" />
            </details>
        </div>
    </div>
</template>
<script>
import TabsTableBlock from './TabsTableBlock.vue';

export default {
    name: 'RelationsTab',
    components: { TabsTableBlock },
    props: {
        relations: {
            type: Array,
            default: () => ([])
        },
        // TODO здесь id а не объект
        editRelation: {
            type: Object,
            default: () => ({})
        }
    },
    data(){
        return {
            search: null,
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
    margin-bottom: none;
}
</style>