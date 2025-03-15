# drawdb-vue2


## Компоненты:

---
- ERDDiargamm
  - `SVGCanvas`
    - `CanvasTableBlock`
  - `ERDTooblar`
    - `RelationsTab`
      - `TabsRelationBlock` 
    - `TablesTab`
      - `TabsTableBlock`
    
---
### ERDDiargamm
Главный компонент-обертка.

Пропсы:
- **tables** (`array`) - массив таблиц/сущностей/блоков. Вид массива:
```javascript
[{
    id: number,
    name: string, // имя таблицы - будет отображаться в заголовке
    comment: string, // комментарий для таблицы
    xAxis: number, // координата по оси X
    yAxis: number, // координата по оси Y
    fields: [ // Список полей в таблице
        {
            id: number,
            name: string, // название поля
            datatype: {id: number, name: string} // тип поля. Для связей рекомендуется использовать MtO, OtM, MtM, OtO
        }]
}]
```
- **relations** (`array`)
- **datatypes** (`array`)

События:

---
TODO:
- настроить генерацию связей между таблицами, 
- добавление полей
- деплой,
- бэк
- стили

## Project setup (Установить зависимости)
```
yarn install
```

### Compiles and hot-reloads for development (Запустить приложение)
```
yarn serve
```

### Compiles and minifies for production (Сборка для деплоя)
```
yarn build
```

### Lints and fixes files (Исправление стилистики кода)
```
yarn lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
