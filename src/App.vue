<template>
  <LeftWrap />
  <MidWrap />
</template>

<script setup>
import LeftWrap from '@/components/leftWrap/index.vue'
import MidWrap from '@/components/midWrap/index.vue'
import { ref, provide } from 'vue'

const state = ref({
  canvasHeight: null,
  blocks: [],
  lines: [],
})

provide('state', state.value)
</script>
