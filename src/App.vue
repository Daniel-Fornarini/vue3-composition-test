<template>
  <VDataTable :sort="sort" @sorted="onSort" :values="value" :columns="columns" :loading="false" >
    <template #column_color="prop">
      <Color :person="prop.value" />
    </template>
  </VDataTable>
</template>

<script setup lang="ts">
  import VDataTable from '@/components/VDataTable.vue';
  import Color from '@/components/Color.vue'
  import {ColumnData} from "@/components/VDataTable.vue";
  import {reactive, ref, watch} from "vue";

  const columns: ColumnData[] = [
    {
      sortable: true,
      sort: '',
      name: 'user',
      class: '',
      default: '-',
      label: 'User'
    },
    {
      sortable: true,
      sort: '',
      name: 'role',
      class: '',
      default: '-',
      label: 'Role'
    },
    {
      sortable: false,
      sort: '',
      name: 'color',
      class: '',
      default: '',
      label: 'Color'
    }
  ];

  let a = ref(0);

  const value: any[] = reactive([
    {
      user: 'Daniel',
      role: 'Dev',
      color: 'red'
    },
    {
      user: 'Federico',
      role: 'Dev',
      color: 'rebeccapurple'
    },
    {
      user: 'Bea',
      role: 'designer',
      color: 'gray'
    },
    {
      user: 'Gabbo',
      role: 'Sales',
      color: 'green'
    }
  ]);

  const sort = ref<string>('');

  function onSort(value: string) {
    sort.value = value;
  }

  watch(() => sort.value, () => {
    if(sort.value[0] === '-') {
      const s = sort.value.slice(1);
      value.sort((a: any, b: any) => {
        if(a[s].toLowerCase() < b[s].toLowerCase()) {
          return 1;
        }
        if(a[s].toLowerCase() > b[s].toLowerCase()) {
          return -1
        }
        return 0;
      });
    } else {
      value.sort((a: any, b: any) => {
        if(a[sort.value].toLowerCase() < b[sort.value].toLowerCase()) {
          return -1;
        }
        if(a[sort.value].toLowerCase() > b[sort.value].toLowerCase()) {
          return 1;
        }
        return 0;
      });
    }
  })
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
