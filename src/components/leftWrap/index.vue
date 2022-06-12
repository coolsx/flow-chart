<template>
  <ul class="leftWrap">
    <template v-for="(node, index) in operatorList" :key="index">
      <li draggable="true" @dragstart="drageStart(node)" @dragend="dragend">
        <Card :title="node.name" style="width: 230px">
          <p>{{ node.description }}</p>
        </Card>
      </li>
    </template>
  </ul>
</template>

<script setup>
import operatorList from './mock.js'
import './index.less'
import { Card } from 'ant-design-vue'
import { inject, nextTick } from 'vue'
const state = inject('state')

//获取画布 用于拖拽控制
let canvas
nextTick(() => {
  canvas = document.querySelector('.canvas')
})

//当前拖拽的node
let curNode

//开始拖拽
function drageStart(node) {
  curNode = node
  canvas.addEventListener('dragenter', dragenter)
  canvas.addEventListener('dragover', dragover)
  canvas.addEventListener('dragleave', dragleave)
  canvas.addEventListener('drop', drop)
}

//结束拖拽
function dragend() {
  curNode = null
  canvas.removeEventListener('dragenter', dragenter)
  canvas.removeEventListener('dragover', dragover)
  canvas.removeEventListener('dragleave', dragleave)
  canvas.removeEventListener('drop', drop)
}

//拖进canvas
function dragenter(event) {
  event.dataTransfer.dropEffect = 'move'
}

//在画布中移动的时候 需要取消默认事件
function dragover(event) {
  event.preventDefault()
}

//离开canvas 改变指针样式
function dragleave(event) {
  event.dataTransfer.dropEffect = 'none'
}

//松手
function drop(event) {
  state.blocks.push({
    id: new Date().getTime(),
    style: {
      left: event.offsetX,
      top: event.offsetY,
      width: null,
      height: null,
    },
    node: curNode,
  })
}
</script>
