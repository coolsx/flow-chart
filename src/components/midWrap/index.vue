<template>
  <div class="midWrap">
    <div
      class="canvas"
      :style="{
        height: state.canvasHeight ? `${state.canvasHeight}px` : 'auto',
      }"
    >
      <template v-for="svg in svgs" :key="svg">
        <SvgPath :path="svg" />
      </template>
      <template v-for="(block, index) in state.blocks" :key="block.name">
        <NodeBlock
          :block="block"
          @mousedown="mousedown(block, $event)"
          @emitClickparam="handleClickParam"
        />
      </template>
    </div>
  </div>
</template>

<script setup>
import './index.less'
import NodeBlock from './componets/nodeBlock'
import SvgPath from './componets/svgPath'
import { notification } from 'ant-design-vue'
import { inject, watch, ref } from 'vue'
const state = inject('state')

let curBlock

//鼠标点下的数据信息
let mouseDownMsg

//点下鼠标
function mousedown(block, event) {
  curBlock = block
  //当前拖拽的block
  const { width: bWidth, height: bHeight, left: bLeft, top: bTop } = block.style
  mouseDownMsg = {
    startX: event.clientX,
    startY: event.clientY,
    startLeft: bLeft,
    startTop: bTop,
    paramsPos: block.node,
  }
  document.addEventListener('mousemove', mousemove)
  document.addEventListener('mouseup', mouseup)
}

//移动鼠标
function mousemove(event) {
  let durX = event.clientX - mouseDownMsg.startX
  let durY = event.clientY - mouseDownMsg.startY
  curBlock.style.left = mouseDownMsg.startLeft + durX
  curBlock.style.top = mouseDownMsg.startTop + durY
}

//松开鼠标
function mouseup() {
  document.removeEventListener('mousemove', mousemove)
  document.removeEventListener('mouseup', mouseup)
}

let twoPoints = { inp: {}, out: {} }
//点击链接点
function handleClickParam(id, param, type, status) {
  if (twoPoints[status].param) {
    notification.error({
      message: '连接错误',
      description: '不能将两个入参/出参连接',
    })
    return false
  }
  if (status == 'inp') {
    if (twoPoints.out.id && twoPoints.out.id == id) {
      notification.error({
        message: '连接错误',
        description: '不能连接同一个算子的出参入参',
      })
      return false
    }
    if (twoPoints.out.type && twoPoints.out.type != type) {
      notification.error({
        message: '连接错误',
        description: '出参与入参类型不一致',
      })
      return false
    }
  }
  if (status == 'out') {
    if (twoPoints.inp.id && twoPoints.inp.id == id) {
      notification.error({
        message: '连接错误',
        description: '不能连接同一个算子的出参入参',
      })
      return false
    }
    if (twoPoints.inp.type && twoPoints.inp.type != type) {
      notification.error({
        message: '连接错误',
        description: '出参与入参类型不一致',
      })
      return false
    }
  }

  twoPoints[status] = {
    param: param,
    type: type,
    id: id,
  }
  //如果存储了两个点 将线点push进state的lines里面
  if (twoPoints.inp.id && twoPoints.out.id) {
    state.lines.push(twoPoints)
    twoPoints = { inp: {}, out: {} }
  }
}

//计算出lines对应的轨迹
let svgs = ref([])
watch(state, newState => {
  svgs.value = []
  newState.lines.forEach(line => {
    getTwoPointXY(line)
  })
})

//获取两个点位置坐标
function getTwoPointXY(line) {
  let inpPoint = { x: null, y: null }
  let outPoint = { x: null, y: null }
  const inputBlock = state.blocks.find(block => {
    if (block.id == line.inp.id) {
      return block
    }
  })
  const outputBlock = state.blocks.find(block => {
    if (block.id == line.out.id) {
      return block
    }
  })
  inpPoint.x =
    inputBlock.node.input[line.inp.param].offsetLeft + inputBlock.style.left
  inpPoint.y =
    inputBlock.node.input[line.inp.param].offsetTop + inputBlock.style.top
  outPoint.x =
    outputBlock.node.output[line.out.param].offsetLeft + outputBlock.style.left
  outPoint.y =
    outputBlock.node.output[line.out.param].offsetTop + outputBlock.style.top
  getBeierPoint(outPoint.x, outPoint.y, inpPoint.x, inpPoint.y)
}

//获取贝塞尔点
function getBeierPoint(startX, startY, endX, endY) {
  const curvature = 0.5
  var hx1 = startX + Math.abs(endX - startX) * curvature
  var hx2 = endX - Math.abs(endX - startX) * curvature

  svgs.value.push(
    `M ${startX} ${startY} C ${hx1} ${startY}  ${hx2} ${endY} ${endX} ${endY}`
  )
}
</script>
