<template>
  <div>
    <ERDDiagramm 
      :tables="tables"
      :relations="relations"
      :datatypes="datatypes"
      @create:table="addTable"
      @create:relation="addRelation"
      @update:relation="updateRelation"
      @delete:relation="deleteRelation"
      @update:table="updateTable"
      @delete:table="deleteTable"
      @create:field="createField"
      @update:field="createField"
    />
  </div>
</template>

<script>
import ErdDiagramm from './components/ERDDiagramm.vue';

export default {
  components: {ERDDiagramm: ErdDiagramm },
  data() {
    return {
      tables: [
        {
          id: 1,
          name: 'User',
          comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
          xAxis: 10,
          yAxis: 25,
          relations: [],
          fields: [
            {
              id: 1,
              name: 'id',
              datatype: {id: 5, name: 'int'},
              nullable: false,
              unique: true, 
              increment: true,
              comment: null
            },
            {
              id: 2,
              datatype: {id: 1, name: 'OtM'},
              name: 'roles',
              nullable: false,
              unique: true, 
              increment: true,
              comment: null
            },
          ]
        },
        {
          id: 2,
          name: 'Role',
          comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
          xAxis: 210,
          yAxis: 225,
          relations: [],
          fields: [
            {
              id: 1,
              name: 'id',
              datatype: {id: 5, name: 'int'},
              nullable: false,
              unique: true, 
              increment: true,
              comment: null
            },
            {
              id: 2,
              name: 'users',
              datatype: {id: 2, name: 'MtO'},
              nullable: false,
              unique: true, 
              increment: true,
              comment: null
            },
          ]
        }
      ],
      relations: [
        {
          id: 1,
          source: {
            id: 1,
            name: 'User',
            comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
            xAxis: 10,
            yAxis: 25,
            relations: [],
            fields: [
              {
                id: 1,
                name: 'id',
                datatype: {id: 5, name: 'int'},
                nullable: false,
                unique: true,
                increment: true,
                comment: null
              },
              {
                id: 2,
                datatype: {id: 1, name: 'OtM'},
                name: 'roles',
                nullable: false,
                unique: true,
                increment: true,
                comment: null
              },
            ]
          },
          target: {
            id: 2,
            name: 'Role',
            comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
            xAxis: 210,
            yAxis: 225,
            relations: [],
            fields: [
              {
                id: 1,
                name: 'id',
                datatype: {id: 5, name: 'int'},
                nullable: false,
                unique: true,
                increment: true,
                comment: null
              },
              {
                id: 2,
                name: 'users',
                datatype: {id: 2, name: 'MtO'},
                nullable: false,
                unique: true,
                increment: true,
                comment: null
              },
            ]
          },
          datatype: {id: 1, name: 'OtM'},
          sourceField: {
            id: 2,
            datatype: {id: 1, name: 'OtM'},
            name: 'roles',
            nullable: false,
            unique: true,
            increment: true,
            comment: null
          },
          targetField: {
            id: 2,
            name: 'users',
            datatype: {id: 2, name: 'MtO'},
            nullable: false,
            unique: true,
            increment: true,
            comment: null
          },
        }
      ],
      datatypes: [
        {id: 1, name: 'OtM'},
        {id: 2, name: 'MtO'},
        {id: 3, name: 'OtO'},
        {id: 4, name: 'MtM'},
        {id: 5, name: 'int'},
        {id: 6, name: 'varchar'},
        {id: 7, name: 'float'},
      ]
    };
  },
  methods: {
    updateTable(table){
      console.log(table);
      const tableIndex = this.tables.findIndex(t => t.id === table.id);
      this.tables[tableIndex] = {...this.tables[tableIndex], ...table};
      console.log(this.tables);
    },
    deleteRelation({ relation }){
      const relationIndex = this.relations.findIndex(rel => rel.id === relation.id);
      this.relations.splice(relationIndex, 1);
    },
    deleteTable({ table }){
      const relationIndex = this.relations.findIndex(rel => rel.target?.id === table.id || rel.source?.id === table.id );
      if (relationIndex !== -1) this.relations.splice(relationIndex, 1);
      const tableIndex = this.tables.findIndex(t => t.id === table);
      console.log('deleting', tableIndex);
      this.tables.splice(tableIndex, 1);
    },
    createField({ table }){
      console.log(table);
      const tableIndex = this.tables.findIndex(t => t.id === table);
      this.tables[tableIndex].fields.push({id: (this.tables[tableIndex].fields?.length || 0 )+ 1, name: 'someField'});
    },
    updateField(data){
      console.log('updateField', data);
      // const tableIndex = this.tables.findIndex(t => t.id === table);
      // this.tables[tableIndex].fields.push({id: (this.tables[tableIndex].fields?.length || 0 )+ 1, name: 'someField'});
    },
    updateRelation(data){
      console.log('updateRelation', data);
    },
    addRelation(data){
      console.log('addRelation', data);
    },
    addTable() {
      this.tables.push({
        id: (this.tables.length || 0) + 1,
        name: `New Table ${(this.tables.length || 0) + 1}`,
        comment: 'Это очень важная таблица, она нужна для существования таблицы ради таблицы',
        xAxis: 10,
        yAxis: 25,
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
  }
};
</script>