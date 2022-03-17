<template>
  <div :style="{ 'width': '30px', height: '30px', 'background-color': color }"></div>
</template>

<script setup lang="ts">
import {computed, defineProps} from "vue";

  const props = defineProps<{
    person: { user: string, role: string, color: string }
  }>()

  const color = computed(() => {
    return props.person.color;
  });
</script>