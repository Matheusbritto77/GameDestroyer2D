<template>
  <div class="universe-wrapper" ref="wrapper">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight" class="universe-canvas"></canvas>
    <div class="hud">
      <span>Posição: ({{ x }}, {{ y }})</span>
      <span>Universo: {{ universeSize }}</span>
    </div>
    <div v-if="isMobile" class="mobile-controls">
      <button @touchstart.prevent="move(0, -1)">⬆️</button>
      <div>
        <button @touchstart.prevent="move(-1, 0)">⬅️</button>
        <button @touchstart.prevent="move(1, 0)">➡️</button>
      </div>
      <button @touchstart.prevent="move(0, 1)">⬇️</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount, computed } from 'vue'
const props = defineProps({
  x: Number,
  y: Number,
  universeSize: Number,
  onMove: Function // função emitida pelo pai
})
const canvas = ref(null)
const wrapper = ref(null)
const canvasWidth = ref(window.innerWidth)
const canvasHeight = ref(window.innerHeight)
const isMobile = computed(() => window.innerWidth < 700)

function resize() {
  canvasWidth.value = window.innerWidth
  canvasHeight.value = window.innerHeight
  draw()
}

function move(dx, dy) {
  if (props.onMove) props.onMove(dx, dy)
}

function draw() {
  if (!canvas.value) return;
  const ctx = canvas.value.getContext('2d')
  ctx.clearRect(0, 0, canvasWidth.value, canvasHeight.value)
  // Efeito de profundidade: gradiente radial
  const grad = ctx.createRadialGradient(
    canvasWidth.value / 2,
    canvasHeight.value / 2,
    80,
    canvasWidth.value / 2,
    canvasHeight.value / 2,
    Math.max(canvasWidth.value, canvasHeight.value) / 1.2
  )
  grad.addColorStop(0, '#23272f')
  grad.addColorStop(1, '#0a0c10')
  ctx.fillStyle = grad
  ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value)
  // Universo (borda com efeito de profundidade)
  ctx.save()
  ctx.strokeStyle = 'rgba(110,231,183,0.5)'
  ctx.lineWidth = 8
  ctx.shadowColor = '#6ee7b7'
  ctx.shadowBlur = 30
  const border = Math.min(canvasWidth.value, canvasHeight.value) * 0.8
  ctx.strokeRect(
    (canvasWidth.value - border) / 2,
    (canvasHeight.value - border) / 2,
    border,
    border
  )
  ctx.restore()
  // Nave do player (centralizada, animada)
  const cx = canvasWidth.value / 2
  const cy = canvasHeight.value / 2
  ctx.save()
  ctx.translate(cx, cy)
  ctx.rotate(Math.atan2(props.y, props.x))
  ctx.beginPath()
  ctx.moveTo(0, -28)
  ctx.lineTo(18, 18)
  ctx.lineTo(-18, 18)
  ctx.closePath()
  ctx.fillStyle = '#34d399'
  ctx.shadowColor = '#34d399'
  ctx.shadowBlur = 18
  ctx.fill()
  ctx.restore()
  // Estrelas de fundo para profundidade
  for (let i = 0; i < 80; i++) {
    const starX = Math.random() * canvasWidth.value
    const starY = Math.random() * canvasHeight.value
    ctx.beginPath()
    ctx.arc(starX, starY, Math.random() * 1.5 + 0.5, 0, 2 * Math.PI)
    ctx.fillStyle = 'rgba(255,255,255,0.15)'
    ctx.fill()
  }
}

// Atualiza o canvas apenas quando as props mudam, sem immediate
watch(
  () => [props.x, props.y, props.universeSize],
  () => {
    draw();
  }
)

onMounted(() => {
  window.addEventListener('resize', resize)
  draw()
})
onBeforeUnmount(() => {
  window.removeEventListener('resize', resize)
})
</script>

<style scoped>
.universe-wrapper {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
  overflow: hidden;
  background: #0a0c10;
}
.universe-canvas {
  width: 100vw;
  height: 100vh;
  display: block;
}
.hud {
  position: absolute;
  top: 18px;
  left: 50%;
  transform: translateX(-50%);
  color: #fff;
  font-size: 1.1rem;
  display: flex;
  gap: 2.5rem;
  background: rgba(24,28,36,0.7);
  padding: 0.5rem 2rem;
  border-radius: 8px;
  pointer-events: none;
  z-index: 2;
}
.mobile-controls {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  z-index: 3;
}
.mobile-controls > div {
  display: flex;
  gap: 2rem;
}
.mobile-controls button {
  background: #23272f;
  color: #fff;
  border: none;
  border-radius: 50%;
  width: 60px;
  height: 60px;
  font-size: 2rem;
  box-shadow: 0 2px 8px #0005;
  margin: 0.2rem;
  cursor: pointer;
  transition: background 0.2s;
}
.mobile-controls button:active {
  background: #34d399;
  color: #181c24;
}
</style> 