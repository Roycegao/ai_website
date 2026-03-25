<template>
  <div class="page">
    <div class="noise"></div>
    <div class="scroll-progress" :style="{ transform: `scaleX(${scrollProgress})` }"></div>

    <header class="topbar glass" :class="{ compact: scrollY > 40 }">
      <div class="brand">maze</div>
      <nav>
        <a v-for="item in navItems" :key="item.href" :href="item.href">{{ item.label }}</a>
      </nav>
      <div class="topbar-actions">
        <button class="btn ghost">Request Demo</button>
        <button class="btn primary mini">Start Free</button>
      </div>
    </header>

    <section class="hero">
      <div class="hero-copy reveal" data-delay="0">
        <p class="eyebrow">Enterprise AI Governance</p>
        <h1>Control, discover and optimize every AI workflow.</h1>
        <p class="sub">
          Built to mirror MazeHQ’s visual language with cinematic gradients, precise spacing,
          elevated typography and narrative scrolling sections.
        </p>
        <div class="cta-row">
          <button class="btn primary">Get Started</button>
          <button class="btn secondary">Watch Overview</button>
        </div>
        <div class="hero-metrics">
          <div><strong>2.8x</strong><span>faster incident response</span></div>
          <div><strong>400+</strong><span>models governed</span></div>
          <div><strong>24/7</strong><span>policy runtime</span></div>
        </div>
      </div>

      <div class="hero-visual reveal" data-delay="80">
        <ParticleGlobe class="hero-globe" />
        <div class="orbit-ring ring-1"></div>
        <div class="orbit-ring ring-2"></div>
      </div>

      <div class="scroll-indicator">Scroll ↓</div>
    </section>

    <section id="product" class="section reveal story-block" data-delay="0">
      <p class="section-kicker">Platform</p>
      <h2>From discovery to deployment, all controls in one place.</h2>
      <p class="section-intro">
        Replace fragmented spreadsheets and ad-hoc checks with a single operating layer across
        teams, agents and model providers.
      </p>
      <div class="card-grid">
        <article
          v-for="(card, i) in cards"
          :key="i"
          class="fancy-card"
          @pointermove="tiltCard"
          @pointerleave="resetCard"
        >
          <div class="card-glint"></div>
          <h3>{{ card.title }}</h3>
          <p>{{ card.desc }}</p>
          <span class="badge">{{ card.badge }}</span>
        </article>
      </div>
    </section>

    <section id="solutions" class="section split reveal story-block" data-delay="30">
      <div>
        <p class="section-kicker">Visibility</p>
        <h2>Workflow intelligence with explainable guardrails.</h2>
        <p>
          Trace prompt lineage, policy overrides, sensitive data exposure and unusual token-spend
          anomalies through one time-synced feed.
        </p>
      </div>
      <div class="stats-panel glass">
        <div class="stat"><strong>99.92%</strong><span>Policy compliance</span></div>
        <div class="stat"><strong>17ms</strong><span>Avg policy check</span></div>
        <div class="stat"><strong>$4.8M</strong><span>Spend optimized</span></div>
      </div>
    </section>

    <section id="resources" class="section timeline reveal story-block" data-delay="40">
      <p class="section-kicker">Narrative Scroll</p>
      <h2>Three-step operating rhythm</h2>
      <div class="timeline-grid">
        <article v-for="(step, idx) in timeline" :key="step.title" class="timeline-card">
          <span class="index">0{{ idx + 1 }}</span>
          <h3>{{ step.title }}</h3>
          <p>{{ step.desc }}</p>
        </article>
      </div>
    </section>

    <section id="pricing" class="section cta-panel reveal" data-delay="0">
      <div>
        <p class="section-kicker">Launch</p>
        <h2>Bring governed AI to production without slowing teams.</h2>
      </div>
      <div class="cta-panel-actions">
        <button class="btn primary">Book Strategy Call</button>
        <button class="btn secondary">View Platform Tour</button>
      </div>
    </section>

    <footer class="footer">
      <p>Designed for demo purposes · MazeHQ visual style clone in Vue</p>
    </footer>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import ParticleGlobe from './components/ParticleGlobe.vue'

const scrollY = ref(0)
const scrollProgress = ref(0)

const navItems = [
  { label: 'Product', href: '#product' },
  { label: 'Solutions', href: '#solutions' },
  { label: 'Resources', href: '#resources' },
  { label: 'Pricing', href: '#pricing' }
]

const cards = [
  { title: 'Model Inventory', desc: 'Unified catalog of models, agents and tools with ownership mapping.', badge: 'Visibility' },
  { title: 'Policy Controls', desc: 'Guardrails with dynamic approvals, thresholding and audit-grade logs.', badge: 'Compliance' },
  { title: 'Risk Scoring', desc: 'Continuous risk scoring by domain, data sensitivity and exposure level.', badge: 'Security' },
  { title: 'Cost Monitoring', desc: 'Granular cost controls and proactive alerts across every workload.', badge: 'FinOps' },
  { title: 'Agent Observability', desc: 'Trace prompts, tools and actions with rich replay timelines.', badge: 'Operations' },
  { title: 'Deployment Gates', desc: 'Automated release checks before production rollouts.', badge: 'Reliability' }
]

const timeline = [
  {
    title: 'Discover',
    desc: 'Map every model, tool and team dependency in a single evolving graph with ownership metadata.'
  },
  {
    title: 'Control',
    desc: 'Apply policy templates and approval rules with real-time checks before high-risk actions execute.'
  },
  {
    title: 'Optimize',
    desc: 'Drive down cost and latency by identifying drift, duplication and underperforming model routes.'
  }
]

function onScroll() {
  scrollY.value = window.scrollY
  const doc = document.documentElement
  const max = doc.scrollHeight - window.innerHeight
  scrollProgress.value = max > 0 ? Math.min(1, window.scrollY / max) : 0
}

function tiltCard(e) {
  const el = e.currentTarget
  if (!el) return
  const rect = el.getBoundingClientRect()
  const px = (e.clientX - rect.left) / rect.width - 0.5
  const py = (e.clientY - rect.top) / rect.height - 0.5
  el.style.transform = `translateY(-6px) rotateX(${(-py * 8).toFixed(2)}deg) rotateY(${(px * 10).toFixed(2)}deg)`
}

function resetCard(e) {
  const el = e.currentTarget
  if (!el) return
  el.style.transform = ''
}

onMounted(() => {
  onScroll()
  window.addEventListener('scroll', onScroll, { passive: true })

  const io = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const delay = Number(entry.target.getAttribute('data-delay') || '0')
          setTimeout(() => entry.target.classList.add('revealed'), delay)
        }
      })
    },
    { threshold: 0.18 }
  )

  document.querySelectorAll('.reveal').forEach((el) => io.observe(el))
})

onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
})
</script>
