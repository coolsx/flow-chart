<template>
  <div
    class="nodeBlock"
    :style="{
      left: `${props.block.style.left}px`,
      top: `${props.block.style.top}px`,
    }"
    ref="refBlock"
  >
    <b>{{ block.node.name }}</b>
    <div class="params">
      <ul class="paramsInp">
        <template v-for="(value, param, index) in block.node.input" :key="index">
          <li
            :data-param="param"
            ref="inpItem"
            @click="clickParam(param, value.type, 'inp')"
          >
            {{ param }} <br />
            <span>({{ value.type }})</span>
          </li>
        </template>
      </ul>
      <ul class="paramsOut">
        <template
          v-for="(value, param, index) in block.node.output"
          :key="index"
        >
          <li
            :data-param="param"
            ref="outItem"
            @click="clickParam(param, value.type, 'out')"
          >
            {{ param }} <br />
            <span>({{ value.type }})</span>
          </li>
        </template>
      </ul>
    </div>
  </div>
</template>

<script setup>
import './index.less'
import { defineProps, defineEmits, onMounted, ref } from 'vue'

const props = defineProps({
  block: null,
})
//存一下block的宽高
const refBlock = ref(null)
const inpItem = ref(null)
const outItem = ref(null)
onMounted(() => {
  let { offsetWidth, offsetHeight } = refBlock.value
  props.block.style.width = offsetWidth
  props.block.style.height = offsetHeight
  //各个输入输出参数相对于block的left与top
  for (let i = 0; i < inpItem.value.length; i++) {
    props.block.node.input[inpItem.value[i].dataset.param].offsetLeft = 0
    props.block.node.input[inpItem.value[i].dataset.param].offsetTop =
      inpItem.value[i].offsetTop+7
  }
  for (let i = 0; i < outItem.value.length; i++) {
    props.block.node.output[outItem.value[i].dataset.param].offsetLeft =
      offsetWidth
    props.block.node.output[outItem.value[i].dataset.param].offsetTop =
      inpItem.value[i].offsetTop+7
  }
})

//点击连接点
const emits = defineEmits(['emitClickparam'])
function clickParam(param, type, status) {
  emits('emitClickparam',props.block.id, param, type, status)
}
</script>
