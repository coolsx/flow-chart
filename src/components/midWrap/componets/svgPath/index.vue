<template>
  <svg>
    <path :d="path"></path>
  </svg>
</template>

<script setup>
import { computed } from 'vue'
import './index.less'
const props = defineProps({
  path: null,
})
</script>
