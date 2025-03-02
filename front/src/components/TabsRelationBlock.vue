<template>
  <div class="relation-block">
    <div class="relation-block-head">
      <span>Имя:</span><span>{{ relationName }}</span>
    </div>

    <div class="relation-block-tables">
      <select
        id="from-select"
        v-model="localValue.source"
        name="tableFrom"
      >
        <option
          value=""
        >
          -- Выберите таблицу --
        </option>
        <option
          v-for="table in tables"
          :key="`from-${table.id}`"
          :value="table.id"
        >
          {{ table.name }}
        </option>
      </select>

      <select
        id="to-select"
        v-model="localValue.target"
        name="tableTo">
        <option value="">
          -- Выберите таблицу --
        </option>
        <option
          v-for="table in tables"
          :key="`to-${table.id}`"
          :value="table.id">
          {{ table.name }}
        </option>
      </select>
    </div>
    <div class="relation-types">
      <label for="relation-types-select">Тип связи</label>
      <select
        id="relation-types-select"
        v-model="localValue.type"
        name="relationType"
      >
        <option value="">-- Выберите тип --</option>
        <option
          v-for="type in datatypes"
          :key="`type-${type.id}`"
          :value="type.id"
        >
          {{ type.name }}
        </option>
      </select> 
    </div>
    <div class="relation-block-tables">
      <select
        id="from-field"
        v-model="localValue.sourceField"
        name="fieldFrom"
      >
        <option value="">-- Выберите таблицу --</option>
        <option
          v-for="table in tables"
          :key="`first-${table.id}`"
          :value="table.id"
        >
          {{ table.name }}
        </option>
      </select>

      <select
        id="to-field"
        v-model="localValue.targetField"
        name="fieldTo"
      >
        <option value="">
          -- Выберите таблицу --
        </option>
        <option
          v-for="table in tables"
          :key="`second-${table.id}`"
          :value="table.id"
        >
          {{ table.name }}
        </option>
      </select>
    </div>
    <button>Удалить</button>
  </div>
</template>
<script>
export default {
  name: 'TabsRelationBlock',
  props: {
    relation: {
      type: Object,
      default: () => ({})
    },
    tables: {
      type: Array,
      default: () => ([])
    },
    datatypes: {
      type: Array,
      default: () => ([])
    },
  },
  data() {
    return {
      localValue: {}
    }
  },
  computed: {
    relationName(){
      return `${this.localValue.from?.name || '(пусто)'} (${this.localValue.type}) ${this.localValue.to?.name || '(пусто)'}`
    }
  },
  watch: {
    relation: {
      immediate: true,
      deep: true,
      handler(value){
        this.localValue = value
      }
    },
    localValue: {
      deep: true,
      handler(value){
        if(JSON.stringify(value) === JSON.stringify(this.relation)) return;
        this.$emit('input', value)
      }
    }
  }
}
</script>
<style scoped>
.relation-block {
    padding: 15px 0;
}

.relation-block-head {
    width: 100%;
    height: 30px;
    display: flex;
    flex-wrap: nowrap;
    gap: 5px;
    align-items: center;
}

summary {
    cursor: pointer;
}

.commet-collapse:open summary {
    padding-bottom: 5px;
}

details {
    border: 1px solid #494949;
    border-radius: 5px;
    padding: 5px;
    margin-bottom: 10px;
}

.relation-block-head span {
    display: inline-block;
    color: #494949;
    font-size: 18px;
    width: 40px;
}

.relation-block-head input {
    outline: none;
    border: none;
    border-bottom: 1px solid #494949;
    transition: border-bottom .2px ease;
    width: 100%;
}

.relation-block-head input:focus {
    border-bottom:  1px solid #3f496a;
}
</style>