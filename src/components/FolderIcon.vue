<template>
  <svg viewBox="0 0 20 20" width="20" height="20"
    @click="$emit('click')" @mouseover="$emit('mouseover')" @mouseout="$emit('mouseout')">
    <path fill-rule="evenodd"
      d="
      M13 6
      H7
      V5
      c0-.5-.5-1-1-1
      H1
      c-.5 0-1 .5-1 1
      V15
      c0 .5.5 1 1 1
      H13
      c.5 0 1-.5 1-1
      V7
      c0-.5-.5-1-1-1
      Z
      M6 6
      H1
      V5
      H6
      V6
      Z">
    </path>
  </svg>
</template>
