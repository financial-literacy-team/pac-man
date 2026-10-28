<script setup lang="ts">
import { ref, nextTick, onMounted, onUnmounted } from 'vue'

const emit = defineEmits<{
  gameOver: [result: { score: number }]
}>()
const canvasRef = ref<HTMLCanvasElement | null>(null)

const gameState = ref('playing')
const score = ref(0)
const cellSize = 40
const cols = 15 
const rows = 10 
let animationFrameId: number


const generateMap = () => {
  return Array.from({ length: rows }, (_, r) =>
    Array.from({ length: cols }, (_, c) => {
      if (r === 0 || r === rows - 1 || c === 0 || c === cols - 1) return 1
      return Math.random() > 0.2 ? 0 : 1 
    })
  )
}

const map = ref<(number)[][]>(generateMap())
const pacman = ref({ x: 80, y: 80, dx: 0, dy: 0, mouthOpen: 0 })

const move = (e: KeyboardEvent) => {
  if (e.key === 'ArrowUp') { pacman.value.dx = 0; pacman.value.dy = -4 }
  if (e.key === 'ArrowDown') { pacman.value.dx = 0; pacman.value.dy = 4 }
  if (e.key === 'ArrowLeft') { pacman.value.dx = -4; pacman.value.dy = 0 }
  if (e.key === 'ArrowRight') { pacman.value.dx = 4; pacman.value.dy = 0 }
}

const restartGame = async () => {
  map.value = generateMap()
  score.value = 0
  gameState.value = 'playing'
  pacman.value = { x: 80, y: 80, dx: 0, dy: 0, mouthOpen: 0 }
  await nextTick()
  gameLoop()
}

const gameLoop = () => {
  if (gameState.value === 'finished') return
  const canvas = canvasRef.value
  const ctx = canvas?.getContext('2d')
  if (!ctx || !canvas) return

  ctx.fillStyle = 'black'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  const nextX = pacman.value.x + pacman.value.dx
  const nextY = pacman.value.y + pacman.value.dy
  const gridX = Math.floor(nextX / cellSize)
  const gridY = Math.floor(nextY / cellSize)

  if (map.value[gridY]?.[gridX] !== 1) {
    pacman.value.x = nextX
    pacman.value.y = nextY
  }

  const currentRow = map.value[gridY]
  if (currentRow?.[gridX] === 0) {
    currentRow[gridX] = 2
    score.value += 10
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      const tile = map.value[r]?.[c]
      if (tile === 1) { ctx.fillStyle = 'blue'; ctx.fillRect(c * cellSize, r * cellSize, cellSize, cellSize) }
      else if (tile === 0) { ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(c * cellSize + 20, r * cellSize + 20, 3, 0, Math.PI * 2); ctx.fill() }
    }
  }

  pacman.value.mouthOpen = (pacman.value.mouthOpen + 0.1) % 1
  ctx.fillStyle = 'yellow'
  ctx.beginPath()
  const angle = pacman.value.mouthOpen < 0.5 ? 0.2 : 0
  ctx.arc(pacman.value.x, pacman.value.y, 15, (angle + 0.2) * Math.PI, (2 - angle - 0.2) * Math.PI)
  ctx.lineTo(pacman.value.x, pacman.value.y)
  ctx.fill()

  if (!map.value.flat().includes(0)) {
    gameState.value = 'finished'
    emit('gameOver', { score: score.value })
    return
  }
  animationFrameId = requestAnimationFrame(gameLoop)
}

onMounted(() => { window.addEventListener('keydown', move); gameLoop() })
onUnmounted(() => { window.removeEventListener('keydown', move); cancelAnimationFrame(animationFrameId) })
</script>

<template>
  <div class="game-wrapper">
    <h1>Pac-man mini game</h1>
    <div class="score">РАХУНОК: {{ score }}</div>
    <canvas v-if="gameState === 'playing'" ref="canvasRef" :width="cols * cellSize" :height="rows * cellSize"></canvas>
    <div v-else class="results">
      <button @click="restartGame">Грати знову!</button>
    </div>

  </div>
</template>

<style scoped>
.game-wrapper { background: black; padding: 20px; display: inline-block; }
.score { color: white; font-family: monospace; font-size: 24px; margin-bottom: 10px; }
.results { color: white; text-align: center; }
</style>