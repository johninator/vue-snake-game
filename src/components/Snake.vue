<template>
  <div class="container">
    <Score :current-score="score" />
    <canvas ref="canvas" width="400" height="400" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import Score from './Score.vue'

const tileSize = 20
let canvas = ref<HTMLCanvasElement>()
let direction = ref('right')
let paused = ref(false)
let gameOver = ref(false)
let score = ref(0)
let speed = ref(100)
let snake = ref([
  { x: tileSize, y: tileSize },
  { x: 0, y: tileSize },
  { x: -tileSize, y: tileSize }
])
let food = ref({ x: 0, y: 0 })

onMounted(() => {
  canvas.value = document.querySelector('canvas') || undefined
  if (!canvas.value) {
    return
  }
  canvas.value.tabIndex = 1
  canvas.value.focus()
  canvas.value.addEventListener('keydown', handleKeyPress)
  generateFood()
  gameLoop()
})

onUnmounted(() => {
  if (!canvas.value) {
    return
  }
  canvas.value.removeEventListener('keydown', handleKeyPress)
})

const restart = () => {
  paused.value = false
  gameOver.value = false
  score.value = 0
  speed.value = 100
  snake.value = [
    { x: tileSize, y: tileSize },
    { x: 0, y: tileSize },
    { x: -tileSize, y: tileSize }
  ]
  food.value = { x: 0, y: 0 }
  direction.value = 'right'
  generateFood()
}

const gameLoop = () => {
  if (!canvas.value) {
    return
  }

  if (!paused.value && !gameOver.value) {
    clearCanvas(canvas.value.getContext('2d')!)
    moveSnake()
    draw()
    gameOver.value = checkCollision()
  }

  if (gameOver.value) {
    alert('Game Over')
    restart()
  }

  setTimeout(gameLoop, speed.value)
}

const draw = () => {
  drawSnake(canvas.value!.getContext('2d')!)
  drawFood(canvas.value!.getContext('2d')!)
}

const drawFood = (ctx: CanvasRenderingContext2D) => {
  ctx.fillStyle = '#f00'
  ctx.fillRect(food.value.x, food.value.y, tileSize, tileSize)
}

const drawSnake = (ctx: CanvasRenderingContext2D) => {
  const gradient = ctx.createLinearGradient(0, 0, canvas.value!.width, canvas.value!.height)
  const gradientLength = snake.value.length * tileSize
  for (let i = 0; i < snake.value.length; i++) {
    const colorStop = i / (snake.value.length - 1)
    const color = `rgba(0, ${Math.round(255 * colorStop)}, 0, 1)`
    gradient.addColorStop(colorStop, color)
  }

  snake.value.forEach((segment, index) => {
    const radius = tileSize / 2
    const angleStart = 0
    const angleEnd = 2 * Math.PI
    ctx.fillStyle = index === 0 ? '#000' : gradient
    ctx.beginPath()
    ctx.arc(segment.x + radius, segment.y + radius, radius, angleStart, angleEnd)
    ctx.fill()
  })
}

const moveSnake = () => {
  const head = { x: snake.value[0].x + getDirection().x, y: snake.value[0].y + getDirection().y }
  snake.value.unshift(head)

  console.log('food: ' + food.value.x + ', ' + food.value.y + ', head: ' + +head.x + ', ' + head.y)

  if (head.x === food.value.x && head.y === food.value.y) {
    score.value += 10
    speed.value -= 5
    generateFood()
  } else {
    snake.value.pop()
  }
}

const getDirection = () => {
  switch (direction.value) {
    case 'up':
      return { x: 0, y: -tileSize }
    case 'down':
      return { x: 0, y: tileSize }
    case 'left':
      return { x: -tileSize, y: 0 }
    case 'right':
      return { x: tileSize, y: 0 }
    default:
      return { x: 0, y: 0 }
  }
}

const generateFood = () => {
  if (!canvas.value) {
    return
  }
  food.value = {
    x: Math.floor(Math.random() * (canvas.value.width / tileSize)) * tileSize,
    y: Math.floor(Math.random() * (canvas.value.height / tileSize)) * tileSize
  }
}

const clearCanvas = (ctx: CanvasRenderingContext2D) => {
  if (!canvas.value) {
    return
  }
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
}

const checkCollision = () => {
  if (!canvas.value) {
    return true
  }
  const head = snake.value[0]
  const hitLeftWall = head.x <= -1
  const hitRightWall = head.x === canvas.value.width
  const hitTopWall = head.y <= -1
  const hitBottomWall = head.y === canvas.value.height
  const hitSelf = snake.value
    .slice(1)
    .some((segment) => segment.x === head.x && segment.y === head.y)

  return hitLeftWall || hitRightWall || hitTopWall || hitBottomWall || hitSelf
}

const handleKeyPress = (event: KeyboardEvent) => {
  switch (event.key) {
    case 'ArrowUp':
    case 'w':
      if (direction.value !== 'down') {
        direction.value = 'up'
      }
      break
    case 'ArrowDown':
    case 's':
      if (direction.value !== 'up') {
        direction.value = 'down'
      }
      break
    case 'ArrowLeft':
    case 'a':
      if (direction.value !== 'right') {
        direction.value = 'left'
      }
      break
    case 'ArrowRight':
    case 'd':
      if (direction.value !== 'left') {
        direction.value = 'right'
      }
      break
    case 'Escape':
      paused.value = !paused.value
      break
    case ' ':
      restart()
      break
  }
}
</script>
<style>
/* .container {
  position: relative;
} */

canvas {
  display: block;
  margin: 0 auto;
  border: 1px solid black;
}
</style>
