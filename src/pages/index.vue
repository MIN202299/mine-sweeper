<script setup lang="ts" generic="T extends any, O extends any">
let WIDTH = 10
let HEIGHT = 10
const isDEV = ref(false)
let minesGenerated = false
let gameOver = false

interface BlockState {
  x: number
  y: number
  mine?: boolean
  adjacentMine: number
  flag: boolean
  reverse: boolean
}

const state = ref<BlockState[][]>([])

function resetState() {
  state.value = Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH }, (_, x) => ({
      x,
      y,
      flag: false,
      adjacentMine: 0,
      reverse: false,
    }) as BlockState),
  )
}

resetState()

function generateMines(initial: BlockState) {
  state.value.forEach((row) => {
    row.forEach((block) => {
      if (Math.abs(block.x - initial.x) < 1)
        return
      if (Math.abs(block.y - initial.y) < 1)
        return
      block.mine = Math.random() < 0.3
    })
  })
  updateNumbers()
}

const directions = [
  [-1, -1],
  [0, -1],
  [1, -1],
  [-1, 0],
  [1, 0],
  [-1, 1],
  [0, 1],
  [1, 1],
]

function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      getAdjacentBlock(block).forEach((sibling) => {
        if (sibling.mine)
          block.adjacentMine++
      })
    })
  })
}

const numberColors = [
  'text-transparent',
  'text-yellow-500',
  'text-blue-500',
  'text-lime-500',
  'text-emerald-500',
  'text-blue-500',
  'text-violet-500',
  'text-pink-500',
  'text-rose-500',
]

function getBlockClass(block: BlockState) {
  if (!block.reverse)
    return 'bg-gray-500/10'
  if (block.mine)
    return 'text-red bg-red-500/10'
  return numberColors[block.adjacentMine]
}

function onClick(block: BlockState) {
  if (gameOver)
    return
  if (!minesGenerated) {
    generateMines(block)
    minesGenerated = true
  }

  if (block.mine) {
    alert('BOOOM!')
    gameOver = true
  }

  block.reverse = true
  expandZero(block)
}

// 展开相邻的0
function expandZero(block: BlockState) {
  if (block.adjacentMine)
    return
  const siblings = getAdjacentBlock(block)
  siblings.forEach((item) => {
    if (!item.reverse) {
      item.reverse = true
      expandZero(item)
    }
  })
}

// 获取临近的block
function getAdjacentBlock(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    return state.value[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function onFlag(block: BlockState) {
  if (block.reverse || gameOver)
    return
  block.flag = !block.flag
}

function checkWin() {
  if (gameOver)
    return
  const flatState = state.value.flat()
  if (flatState.filter(block => !block.reverse).every(block => block.mine)) {
    alert('you win')
    gameOver = true
  }
}

function newGame(width = WIDTH, height = HEIGHT) {
  WIDTH = width
  HEIGHT = height
  minesGenerated = false
  gameOver = false
  resetState()
}

watchEffect(checkWin)
</script>

<template>
  <div>
    <h1 mb-5>
      MineSweeper
    </h1>
    <div flex="~ gap-2" justify-center>
      <button btn @click="newGame()">
        新游戏
      </button>
      <button btn @click="newGame(5, 5)">
        简单
      </button>
      <button btn @click="newGame(10, 10)">
        中等
      </button>
      <button btn @click="newGame(20, 20)">
        地狱
      </button>
      <button btn @click="isDEV = !isDEV">
        作弊
      </button>
    </div>
    <div p-5>
      <div
        v-for="row, y in state"
        :key="y"
        flex="~"
        items-center justify-center
      >
        <button
          v-for="block, x in row"
          :key="x"
          m-1px
          h-10 w-10
          flex="~"
          items-center justify-center
          border="1 gray-400/20"
          hover="bg-gray/40"
          :class="getBlockClass(block)"
          @click="onClick(block)"
          @contextmenu.prevent="onFlag(block)"
        >
          <template v-if="block.flag">
            <div i-mdi-flag text-red />
          </template>
          <template v-else-if="block.reverse || isDEV">
            <div v-if="block.mine" i-mdi:mine />
            <div v-else>
              {{ block.adjacentMine }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
