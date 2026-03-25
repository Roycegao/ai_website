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
let dpr = 1
let w = 0
let h = 0
let cx = 0
let cy = 0
let radius = 0
let rotY = 0
let rotX = -0.2
let velY = 0
let velX = 0
let dragging = false
let lastX = 0
let lastY = 0

const points = []
const stars = []
const pointCount = 1300
const starCount = 180

function createFibonacciSphere(count) {
  points.length = 0
  const goldenAngle = Math.PI * (3 - Math.sqrt(5))

  for (let i = 0; i < count; i++) {
    const y = 1 - (i / (count - 1)) * 2
    const r = Math.sqrt(1 - y * y)
    const theta = goldenAngle * i

    points.push({
      x: Math.cos(theta) * r,
      y,
      z: Math.sin(theta) * r,
      s: Math.random() * 1.4 + 0.5
    })
  }
}

function createStars() {
  stars.length = 0
  for (let i = 0; i < starCount; i++) {
    stars.push({
      x: Math.random(),
      y: Math.random(),
      r: Math.random() * 1.4 + 0.2,
      a: Math.random() * 0.6 + 0.12,
      tw: Math.random() * 0.012 + 0.006
    })
  }
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
  radius = Math.min(w, h) * 0.32
}

function projectPoint(p) {
  const yRot = rotateY(p.x, p.z, rotY)
  const xRot = rotateX(p.y, yRot.z, rotX)

  const depth = (xRot.z + 1.5) / 2.5
  const perspective = 0.58 + depth * 0.82

  return {
    sx: cx + yRot.x * radius * perspective,
    sy: cy + xRot.y * radius * perspective,
    depth,
    size: p.s * perspective,
    front: xRot.z > -0.1
  }
}

function drawStars(time) {
  for (let i = 0; i < stars.length; i++) {
    const s = stars[i]
    const alpha = s.a + Math.sin(time * s.tw + i) * 0.16
    ctx.fillStyle = `rgba(160, 220, 255, ${Math.max(0.08, alpha)})`
    ctx.beginPath()
    ctx.arc(s.x * w, s.y * h, s.r, 0, Math.PI * 2)
    ctx.fill()
  }
}

function drawSphereBody() {
  const gradient = ctx.createRadialGradient(
    cx - radius * 0.34,
    cy - radius * 0.36,
    radius * 0.2,
    cx,
    cy,
    radius * 1.05
  )

  gradient.addColorStop(0, 'rgba(38, 127, 255, 0.34)')
  gradient.addColorStop(0.5, 'rgba(20, 79, 180, 0.2)')
  gradient.addColorStop(1, 'rgba(9, 28, 64, 0.08)')

  ctx.fillStyle = gradient
  ctx.beginPath()
  ctx.arc(cx, cy, radius, 0, Math.PI * 2)
  ctx.fill()
}

function drawGraticule() {
  ctx.lineWidth = 1
  for (let lat = -60; lat <= 60; lat += 20) {
    const t = (lat * Math.PI) / 180
    ctx.beginPath()
    let started = false

    for (let lon = 0; lon <= 360; lon += 6) {
      const p = {
        x: Math.cos(t) * Math.cos((lon * Math.PI) / 180),
        y: Math.sin(t),
        z: Math.cos(t) * Math.sin((lon * Math.PI) / 180)
      }
      const pr = projectPoint(p)
      if (pr.front) {
        if (!started) {
          ctx.moveTo(pr.sx, pr.sy)
          started = true
        } else {
          ctx.lineTo(pr.sx, pr.sy)
        }
      }
    }

    ctx.strokeStyle = 'rgba(121, 206, 255, 0.16)'
    ctx.stroke()
  }

  for (let lon = 0; lon < 180; lon += 20) {
    ctx.beginPath()
    let started = false

    for (let lat = -88; lat <= 88; lat += 4) {
      const t = (lat * Math.PI) / 180
      const p = {
        x: Math.cos(t) * Math.cos((lon * Math.PI) / 180),
        y: Math.sin(t),
        z: Math.cos(t) * Math.sin((lon * Math.PI) / 180)
      }
      const pr = projectPoint(p)
      if (pr.front) {
        if (!started) {
          ctx.moveTo(pr.sx, pr.sy)
          started = true
        } else {
          ctx.lineTo(pr.sx, pr.sy)
        }
      }
    }

    ctx.strokeStyle = 'rgba(121, 206, 255, 0.1)'
    ctx.stroke()
  }
}

function drawParticleMesh() {
  const projected = points.map(projectPoint).sort((a, b) => a.depth - b.depth)

  for (let i = 0; i < projected.length; i++) {
    const p = projected[i]

    for (let j = i + 1; j < Math.min(i + 14, projected.length); j++) {
      const q = projected[j]
      const dx = p.sx - q.sx
      const dy = p.sy - q.sy
      const dist = Math.sqrt(dx * dx + dy * dy)
      if (dist > 30) continue

      const alpha = (1 - dist / 30) * 0.22 * Math.min(p.depth, q.depth)
      if (alpha <= 0) continue

      ctx.strokeStyle = `rgba(130, 218, 255, ${alpha})`
      ctx.lineWidth = 0.7
      ctx.beginPath()
      ctx.moveTo(p.sx, p.sy)
      ctx.lineTo(q.sx, q.sy)
      ctx.stroke()
    }

    ctx.fillStyle = `rgba(166, 236, 255, ${0.2 + p.depth * 0.84})`
    ctx.beginPath()
    ctx.arc(p.sx, p.sy, p.size, 0, Math.PI * 2)
    ctx.fill()
  }
}

function drawAtmosphere() {
  const glow = ctx.createRadialGradient(cx, cy, radius * 0.68, cx, cy, radius * 1.35)
  glow.addColorStop(0, 'rgba(57, 149, 255, 0.22)')
  glow.addColorStop(1, 'rgba(57, 149, 255, 0)')

  ctx.fillStyle = glow
  ctx.beginPath()
  ctx.arc(cx, cy, radius * 1.35, 0, Math.PI * 2)
  ctx.fill()
}

function drawFrame(time) {
  if (!ctx) return

  ctx.clearRect(0, 0, w, h)
  drawStars(time)

  rotY += velY + 0.0025
  rotX += velX

  velY *= 0.94
  velX *= 0.92
  rotX = Math.max(-0.75, Math.min(0.75, rotX))

  drawAtmosphere()
  drawSphereBody()
  drawGraticule()
  drawParticleMesh()

  raf = requestAnimationFrame(drawFrame)
}

function pointerDown(e) {
  dragging = true
  lastX = e.clientX
  lastY = e.clientY
}

function pointerMove(e) {
  if (!dragging) {
    const rect = wrapRef.value?.getBoundingClientRect()
    if (!rect) return
    const mx = (e.clientX - rect.left) / rect.width - 0.5
    const my = (e.clientY - rect.top) / rect.height - 0.5
    velY += mx * 0.0014
    velX += my * 0.0008
    return
  }

  const dx = e.clientX - lastX
  const dy = e.clientY - lastY
  lastX = e.clientX
  lastY = e.clientY

  velY = dx * 0.00055
  velX = dy * 0.00042
}

function pointerUp() {
  dragging = false
}

onMounted(() => {
  createFibonacciSphere(pointCount)
  createStars()
  resize()
  drawFrame(0)

  window.addEventListener('resize', resize)
  wrapRef.value?.addEventListener('pointerdown', pointerDown)
  wrapRef.value?.addEventListener('pointermove', pointerMove)
  window.addEventListener('pointerup', pointerUp)
})

onUnmounted(() => {
  if (raf) cancelAnimationFrame(raf)

  window.removeEventListener('resize', resize)
  wrapRef.value?.removeEventListener('pointerdown', pointerDown)
  wrapRef.value?.removeEventListener('pointermove', pointerMove)
  window.removeEventListener('pointerup', pointerUp)
})
</script>
