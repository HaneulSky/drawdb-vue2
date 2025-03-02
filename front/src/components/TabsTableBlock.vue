<template>
  <div class="table-block">
    <div class="table-block-head">
      <span>Имя:</span><input :value="table.name" >
    </div>
    <div
      v-for="field in table.fields"
      :key="field.id"
      class="table-block-field"
    >
      <span>
        {{ field.name }}
      </span>
      <span>
        {{ field.type }} 
      </span>
    </div>
    <div>
      <details class="comment-collapse">
        <summary>Комментарий</summary>
        <textarea :value="table.comment" />
      </details>
    </div>
    <button>Добавить поле</button>
  </div>
</template>
<script>
export default {
  name: 'TabsTableBlock',
  props: {
    table:{
      type: Object,
      default: () => ({})
    }
  },
  data(){
    return {
            
    }
  }
}
</script>
<style scoped>
.table-block {
    padding: 15px 0;
}

.table-block-head {
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

.table-block-head span {
    display: inline-block;
    color: #494949;
    font-size: 18px;
    width: 40px;
}

.table-block-head input {
    outline: none;
    border: none;
    border-bottom: 1px solid #494949;
    transition: border-bottom .2px ease;
    width: 100%;
}

.table-block-head input:focus {
    border-bottom:  1px solid #3f496a;
}
</style>