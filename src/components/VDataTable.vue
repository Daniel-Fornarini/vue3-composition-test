<template>
  <div class="data-table-container">
    <div class="table-responsive" :class="{'content-loading': props.loading}">
      <table class="table" :class="props.tableClasses">
        <thead class="thead-light">
        <tr>
          <th v-for="column in props.columns" :key="column.name" :class="getColumnClass(column)">
            <a href="#" @click.prevent="sortBy(column)" v-if="props.sortable && column.sortable !== false">
              {{ column.label }}
              <template v-if="isSortedBy(column)">
                <slot v-if="isSortedByDesc(column)" name="sort-icon-down">⇩</slot>
                <slot v-else name="sort-icon-up">⇧</slot>
              </template>
            </a>
            <template v-else>
              {{ column.label }}
            </template>
          </th>
          <th v-if="hasActions()" class="col-actions">
            <slot name="actions-label">Actions</slot>
          </th>
        </tr>
        </thead>
        <tbody>
        <template v-if="props.values.length">
          <tr v-for="value in props.values" :key="value.id">
            <td v-for="column in props.columns" :key="column.name" :class="getColumnClass(column)" @click="select(value)">
              <template v-if="hasSlot(column)">
                <slot :name="getSlotName(column)" :value="value"></slot>
              </template>
              <template v-else>
                {{ getValue(column, value) }}
              </template>
            </td>
            <td class="col-actions" v-if="hasActions()">
              <slot name="actions" :value="value"></slot>
            </td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td :colspan="columnsTotal" class="col-no-results">
              <span v-if="!props.loading">
                <slot name="no-results-label">No result</slot>
              </span>
            </td>
          </tr>
        </template>
        </tbody>
      </table>
      <div class="loading-label" v-if="props.loading">
        <slot name="loading">Loading...</slot>
      </div>
    </div>
    <slot></slot>
  </div>
</template>


<script setup lang="ts">
  import {computed, defineProps, reactive, watch, withDefaults, defineEmits, useSlots} from "vue";


  export interface  ColumnData {
    sortable: boolean;
    sort: string;
    name: string;
    class: string;
    default: string;
    label: string;
  }

  interface Props {
    columns: ColumnData[],
    tableClasses?: string,
    values: any[],
    sort?: string,
    sortable: boolean,
    loading: boolean
  }

  const props = withDefaults(defineProps<Props>(), {
    tableClasses: 'table-sm table-hover',
    sort: '',
    sortable: true,
    loading: true
  });

  const emit = defineEmits(['selected', 'sorted']);
  const slots = useSlots();

  let _sort: Map<string, string> = reactive(new Map<string, string>());

  parseSort();

  const columnsTotal = computed(() => {
      let total = props.columns.length;
      if(hasActions()) {
        total++;
      }
      return total;
  });

  watch(() => props.sort, () => {
    parseSort();
  });

  function select(value: any) {
    emit('selected', value);
  }

  function parseSort() {
    _sort = new Map<string, string>();
    props.sort.split(',').map(item => {
      const order = item[0] === '-' ? '-' : '';
      const key = order ? item.substr(1) : item;
      _sort.set(key, order);
    });
  }

  function sortBy(column: ColumnData) {
    if(!props.sortable) {
      return;
    }

    const name = column.sort || column.name;
    let order = _sort.get(name);

    if(order == null) {
      order = '';
    } else {
      order = order === '-' ? '' : '-';
    }

    const sort = `${order}${name}`;
    emit('sorted', sort);
  }

  function isSortedBy(column: ColumnData) {
    const name = column.sort || column.name;
    return _sort.get(name) != null;
  }

  function isSortedByDesc(column: ColumnData) {
    const name = column.sort || column.name;
    return _sort.get(name) === '-';
  }

  function getColumnClass(column: ColumnData) {
    const columnClass: { [key: string]: boolean } = {
      'col-oder-by': isSortedBy(column)
    };

    columnClass[`col-data__${column.name}`] = true;

    if (column.class) {
      columnClass[column.class] = true;
    }

    return columnClass;
  }

  function getValue(column: ColumnData, value: any) {
    if (column.name.indexOf('.') !== -1) {
      const methodChain = column.name.split('.');
      let result = value;
      for (const method of methodChain) {
        result = result ? result[method] : null;
      }
      return result || column.default;
    }
    return value[column.name] || column.default;
  }

  function hasActions() {
    return slots.actions != null;
  }

  function getSlotName(column: ColumnData) {
    return `column_${column.name}`;
  }

  function hasSlot(column: ColumnData) {
    const slotName = getSlotName(column);
    return slots[slotName] != null;
  }
</script>

<style lang="scss" scoped>
.table-responsive {
  min-height: 320px;
}
.col-no-results {
  height: 270px;
  vertical-align: middle;
  border-bottom: 0;
}
</style>
