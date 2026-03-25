<template>
  <canvas ref="canvasRef" class="scroll-particle-canvas" aria-hidden="true"></canvas>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

const canvasRef = ref(null)
let ctx = null
let raf = null
let dpr = 1
let w = 0
let h = 0
let scrollFactor = 0
let mouseX = 0.5
let mouseY = 0.5

const particles = []
const count = 240

function seed() {
  particles.length = 0
  for (let i = 0; i < count; i++) {
    particles.push({
      x: Math.random(),
      y: Math.random(),
      vx: (Math.random() - 0.5) * 0.0002,
      vy: (Math.random() - 0.5) * 0.0002,
      s: Math.random() * 1.7 + 0.35,
      a: Math.random() * 0.5 + 0.08
    })
  }
}

function resize() {
  const canvas = canvasRef.value
  if (!canvas) return

  dpr = window.devicePixelRatio || 1
  w = window.innerWidth
  h = window.innerHeight

  canvas.width = Math.floor(w * dpr)
  canvas.height = Math.floor(h * dpr)
  canvas.style.width = `${w}px`
  canvas.style.height = `${h}px`

  ctx = canvas.getContext('2d')
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
}

function onScroll() {
  const max = document.documentElement.scrollHeight - window.innerHeight
  const p = max > 0 ? window.scrollY / max : 0
  scrollFactor = Math.min(1, Math.max(0, p))
}

function onPointer(e) {
  mouseX = e.clientX / w
  mouseY = e.clientY / h
}

function draw(time) {
  if (!ctx) return
  ctx.clearRect(0, 0, w, h)

  for (let i = 0; i < particles.length; i++) {
    const p = particles[i]

    p.x += p.vx + (mouseX - 0.5) * 0.00035
    p.y += p.vy - scrollFactor * 0.0024 - (0.5 - mouseY) * 0.00028

    if (p.x > 1.02) p.x = -0.02
    if (p.x < -0.02) p.x = 1.02
    if (p.y > 1.02) p.y = -0.02
    if (p.y < -0.02) p.y = 1.02

    const x = p.x * w
    const y = p.y * h
    const twinkle = Math.sin(time * 0.001 + i) * 0.16
    const alpha = Math.max(0.06, p.a + twinkle)

    ctx.fillStyle = `rgba(136, 221, 255, ${alpha})`
    ctx.beginPath()
    ctx.arc(x, y, p.s + scrollFactor * 0.45, 0, Math.PI * 2)
    ctx.fill()

    if (i % 12 === 0) {
      const tail = 14 + scrollFactor * 36
      ctx.strokeStyle = `rgba(124, 211, 255, ${0.07 + scrollFactor * 0.14})`
      ctx.lineWidth = 1
      ctx.beginPath()
      ctx.moveTo(x, y)
      ctx.lineTo(x, y + tail)
      ctx.stroke()
    }
  }

  raf = requestAnimationFrame(draw)
}

onMounted(() => {
  seed()
  resize()
  onScroll()
  raf = requestAnimationFrame(draw)

  window.addEventListener('resize', resize)
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('pointermove', onPointer, { passive: true })
})

onUnmounted(() => {
  if (raf) cancelAnimationFrame(raf)
  window.removeEventListener('resize', resize)
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('pointermove', onPointer)
})
</script>
