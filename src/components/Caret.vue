<template>
  <svg height="20" width="20"
    @click="$emit('click')" @mouseover="$emit('mouseover')" @mouseout="$emit('mouseout')">
    <path stroke="currentColor" stroke-width="1" stroke-linecap="round" d="M8 6 12 10M12 10 8 14" />
  </svg>
</template>

<style>
.caret-open {
  transform: rotate(90deg);
}
</style>
