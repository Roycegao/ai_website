<template>
  <div class="globe-wrap" ref="wrapRef">
    <canvas ref="canvasRef"></canvas>
    <div class="glow"></div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

const canvasRef = ref(null)
const wrapRef = ref(null)

let raf = null
let ctx = null
let points = []
let w = 0
let h = 0
let dpr = 1
let cx = 0
let cy = 0
let globeR = 0
let rot = 0
let mouseBiasX = 0
let mouseBiasY = 0

const totalPoints = 1000

function randomSpherePoint() {
  const u = Math.random()
  const v = Math.random()
  const theta = 2 * Math.PI * u
  const phi = Math.acos(2 * v - 1)

  const x = Math.sin(phi) * Math.cos(theta)
  const y = Math.sin(phi) * Math.sin(theta)
  const z = Math.cos(phi)

  return { x, y, z, size: Math.random() * 1.5 + 0.4 }
}

function initPoints() {
  points = Array.from({ length: totalPoints }, randomSpherePoint)
}

function resize() {
  const canvas = canvasRef.value
  const wrap = wrapRef.value
  if (!canvas || !wrap) return

  dpr = window.devicePixelRatio || 1
  w = wrap.clientWidth
  h = wrap.clientHeight
  canvas.width = Math.floor(w * dpr)
  canvas.height = Math.floor(h * dpr)
  canvas.style.width = `${w}px`
  canvas.style.height = `${h}px`

  ctx = canvas.getContext('2d')
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)

  cx = w / 2
  cy = h / 2
  globeR = Math.min(w, h) * 0.34
}

function rotateY(x, z, angle) {
  const c = Math.cos(angle)
  const s = Math.sin(angle)
  return { x: x * c - z * s, z: x * s + z * c }
}

function rotateX(y, z, angle) {
  const c = Math.cos(angle)
  const s = Math.sin(angle)
  return { y: y * c - z * s, z: y * s + z * c }
}

function drawConnection(a, b) {
  const dx = a.sx - b.sx
  const dy = a.sy - b.sy
  const dist = Math.sqrt(dx * dx + dy * dy)
  if (dist > 42) return

  const alpha = Math.max(0, (1 - dist / 42) * 0.24)
  ctx.strokeStyle = `rgba(120, 210, 255, ${alpha})`
  ctx.lineWidth = 0.8
  ctx.beginPath()
  ctx.moveTo(a.sx, a.sy)
  ctx.lineTo(b.sx, b.sy)
  ctx.stroke()
}

function drawAtmosphere() {
  const grad = ctx.createRadialGradient(cx, cy, globeR * 0.6, cx, cy, globeR * 1.22)
  grad.addColorStop(0, 'rgba(57, 149, 255, 0.14)')
  grad.addColorStop(1, 'rgba(57, 149, 255, 0)')
  ctx.fillStyle = grad
  ctx.beginPath()
  ctx.arc(cx, cy, globeR * 1.22, 0, Math.PI * 2)
  ctx.fill()
}

function frame() {
  if (!ctx) return
  rot += 0.0034

  ctx.clearRect(0, 0, w, h)
  drawAtmosphere()

  const projected = []
  for (let i = 0; i < points.length; i++) {
    const p = points[i]

    const ry = rotateY(p.x, p.z, rot + mouseBiasX * 0.15)
    const rx = rotateX(p.y, ry.z, mouseBiasY * 0.1)

    const depth = (rx.z + 1.4) / 2.4
    const perspective = 0.65 + depth * 0.7

    const sx = cx + ry.x * globeR * perspective
    const sy = cy + rx.y * globeR * perspective

    projected.push({ sx, sy, depth, size: p.size * perspective })
  }

  projected.sort((a, b) => a.depth - b.depth)

  for (let i = 0; i < projected.length; i++) {
    const p = projected[i]

    for (let j = i + 1; j < Math.min(projected.length, i + 20); j++) {
      drawConnection(p, projected[j])
    }

    const alpha = 0.2 + p.depth * 0.8
    ctx.fillStyle = `rgba(151, 232, 255, ${alpha})`
    ctx.beginPath()
    ctx.arc(p.sx, p.sy, p.size, 0, Math.PI * 2)
    ctx.fill()
  }

  raf = requestAnimationFrame(frame)
}

function onPointerMove(e) {
  const rect = wrapRef.value?.getBoundingClientRect()
  if (!rect) return

  const mx = (e.clientX - rect.left) / rect.width - 0.5
  const my = (e.clientY - rect.top) / rect.height - 0.5

  mouseBiasX = mx * 2
  mouseBiasY = my * 2
}

onMounted(() => {
  initPoints()
  resize()
  frame()

  window.addEventListener('resize', resize)
  wrapRef.value?.addEventListener('pointermove', onPointerMove)
})

onUnmounted(() => {
  if (raf) cancelAnimationFrame(raf)
  window.removeEventListener('resize', resize)
  wrapRef.value?.removeEventListener('pointermove', onPointerMove)
})
</script>
