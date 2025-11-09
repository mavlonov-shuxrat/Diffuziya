<template>
  <div class="diffusion-wrapper">
    <canvas ref="canvas" class="diffusion-canvas"></canvas>
    <div class="controls">
      <label
        >Температура: <strong>{{ temperature }}</strong></label
      >
      <input type="range" min="0" max="200" v-model="temperature" />
      <label
        >Кол-во частиц: <strong>{{ numParticles }}</strong></label
      >
      <input
        type="range"
        min="50"
        max="1000"
        step="10"
        v-model="numParticles"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from "vue";

const canvas = ref(null);
const ctx = ref(null);

const temperature = ref(60);
const numParticles = ref(300);

let particles = [];
let rafId = null;
let width = 0;
let height = 0;

class Particle {
  constructor(x, y, radius, color, type) {
    this.x = x;
    this.y = y;
    this.radius = radius;
    this.color = color;
    this.type = type; // for e.g., different metals
    this.vx = 0;
    this.vy = 0;
  }

  step(dt, thermalFactor) {
    // Brownian-like random walk scaled with temperature
    const scale = thermalFactor * (0.5 + Math.random());
    this.vx += (Math.random() - 0.5) * scale;
    this.vy += (Math.random() - 0.5) * scale;

    // simple damping to keep speeds reasonable
    this.vx *= 0.98;
    this.vy *= 0.98;

    this.x += this.vx * dt;
    this.y += this.vy * dt;

    // boundary reflection
    if (this.x < this.radius) {
      this.x = this.radius;
      this.vx *= -0.6;
    }
    if (this.x > width - this.radius) {
      this.x = width - this.radius;
      this.vx *= -0.6;
    }
    if (this.y < this.radius) {
      this.y = this.radius;
      this.vy *= -0.6;
    }
    if (this.y > height - this.radius) {
      this.y = height - this.radius;
      this.vy *= -0.6;
    }
  }

  draw(c) {
  c.beginPath();
  c.shadowBlur = 12;
  c.shadowColor = this.color;
  c.fillStyle = this.color;
  c.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
  c.fill();
  c.shadowBlur = 0; // сброс, чтобы не светился фон
}

}

function initParticles() {
  particles = [];
  const half = Math.floor(numParticles.value / 2);
  const r = Math.max(2, Math.min(4, Math.sqrt(width * height) / 200));

  // left half - metal A (e.g., Gold-ish), right half - metal B (e.g., Silver-ish)
  for (let i = 0; i < half; i++) {
    const x = Math.random() * (width * 0.45);
    const y = Math.random() * height;
    particles.push(new Particle(x, y, r, "rgba(255,0,0,0.95)", "A")); // красный
  }
  for (let i = 0; i < numParticles.value - half; i++) {
    const x = width * 0.55 + Math.random() * (width * 0.45);
    const y = Math.random() * height;
    particles.push(new Particle(x, y, r, "rgba(0,0,255,0.95)", "B")); // синий
  }
}

function resizeCanvas() {
  const dpr = window.devicePixelRatio || 1;
  const el = canvas.value;
  if (!el) return;
  width = el.clientWidth;
  height = el.clientHeight;
  el.width = Math.floor(width * dpr);
  el.height = Math.floor(height * dpr);
  ctx.value = el.getContext("2d");
  ctx.value.setTransform(dpr, 0, 0, dpr, 0, 0);
}

let lastTs = 0;
function step(ts) {
  if (!lastTs) lastTs = ts;
  const dt = Math.min(32, ts - lastTs) / 16; // normalize to ~60fps units
  lastTs = ts;

  // thermal factor: convert temperature slider to motion scale
  const thermalFactor = Math.max(0.1, temperature.value / 60);

  // simulation update
  for (let p of particles) p.step(dt, thermalFactor);

  // optional simple short-range mixing effect: slight attraction if different types overlap
  // this emulates initial stages of interdiffusion on mesoscopic scale
  const k = 0.01 * thermalFactor;
  for (let i = 0; i < particles.length; i++) {
    for (let j = i + 1; j < particles.length; j++) {
      const a = particles[i];
      const b = particles[j];
      const dx = b.x - a.x;
      const dy = b.y - a.y;
      const dist2 = dx * dx + dy * dy;
      const minDist = (a.radius + b.radius) * 1.0;
      if (dist2 > 0 && dist2 < minDist * minDist) {
        const dist = Math.sqrt(dist2);
        const nx = dx / dist;
        const ny = dy / dist;
        // small displacement to avoid overlap
        const overlap = (minDist - dist) * 0.5;
        a.x -= nx * overlap;
        a.y -= ny * overlap;
        b.x += nx * overlap;
        b.y += ny * overlap;

        // if different types, small exchange velocity -> enhances mixing at contact
        if (a.type !== b.type) {
          a.vx -= k * nx;
          a.vy -= k * ny;
          b.vx += k * nx;
          b.vy += k * ny;
        }
      }
    }
  }

  // render
  const c = ctx.value;
  c.clearRect(0, 0, width, height);

  // draw faint background gradient to emphasize center mixing
  const grad = c.createLinearGradient(0, 0, width, 0);
  grad.addColorStop(0, "rgba(10,10,10,0.98)");
  grad.addColorStop(0.45, "rgba(18,18,18,0.98)");
  grad.addColorStop(0.55, "rgba(18,18,18,0.98)");
  grad.addColorStop(1, "rgba(10,10,10,0.98)");
  c.fillStyle = grad;
  c.fillRect(0, 0, width, height);

  for (let p of particles) p.draw(c);

  rafId = requestAnimationFrame(step);
}

onMounted(() => {
  // set canvas size to container
  const el = canvas.value;
  el.style.width = "100%";
  el.style.height = "60vh";
  resizeCanvas();
  initParticles();
  window.addEventListener("resize", () => {
    resizeCanvas();
    initParticles();
  });
  rafId = requestAnimationFrame(step);
});

onBeforeUnmount(() => {
  if (rafId) cancelAnimationFrame(rafId);
});

watch(numParticles, () => {
  initParticles();
});

watch(temperature, () => {
  // no heavy action required; animation uses value dynamically
});
</script>

<style scoped>
.diffusion-wrapper {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.diffusion-canvas {
  width: 100%;
  height: 60vh;
  background: #0b0b0b;
  border-radius: 8px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.6);
}
.controls {
  display: flex;
  gap: 16px;
  align-items: center;
}
.controls label {
  color: #ddd;
  font-size: 14px;
}
.controls input[type="range"] {
  width: 160px;
}
</style>
