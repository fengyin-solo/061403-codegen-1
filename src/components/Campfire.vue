<template>
  <div class="campfire-container">
    <div class="campfire-scene">
      <canvas ref="canvasRef" :width="canvasSize" :height="canvasSize"></canvas>
      <div class="buildings-overlay">
        <div 
          v-for="slot in buildingSlots" 
          :key="slot.id"
          class="building-slot"
          :class="[
            slot.position,
            { occupied: buildings[slot.buildingId] > 0 },
            { 'slot-warm': slot.buildingId === 'windWall' },
            { 'slot-food': slot.buildingId === 'storageCellar' },
            { 'slot-event': slot.buildingId === 'watchtower' },
            { 'pulse-warm': slot.buildingId === 'windWall' && !isDay && buildings[slot.buildingId] > 0 },
            { 'pulse-food': slot.buildingId === 'storageCellar' && isDay && buildings[slot.buildingId] > 0 },
            { 'pulse-event': slot.buildingId === 'watchtower' && buildings[slot.buildingId] > 0 }
          ]"
          :style="slot.style"
        >
          <div v-if="buildings[slot.buildingId] > 0" class="building-display">
            <span class="building-icon-lg">{{ slot.icon }}</span>
            <span class="building-badge">x{{ buildings[slot.buildingId] }}</span>
          </div>
          <div v-else class="slot-empty">
            <span class="slot-placeholder">+</span>
          </div>
          <div class="slot-label">{{ slot.name }}</div>
        </div>
      </div>
      <div class="effect-indicators" v-if="hasAnyBuilding">
        <div v-if="warmReduction > 0" class="effect-badge warm-effect" :title="`热量消耗减少 ${Math.round(warmReduction * 100)}%`">
          <span class="effect-icon">🛡️</span>
          <span class="effect-text">-{{ Math.round(warmReduction * 100) }}%</span>
        </div>
        <div v-if="foodBonus > 0" class="effect-badge food-effect" :title="`每日产出 ${foodBonus} 食物`">
          <span class="effect-icon">🍖</span>
          <span class="effect-text">+{{ foodBonus }}/天</span>
        </div>
        <div v-if="stormReduction > 0" class="effect-badge event-effect" :title="`暴风雪概率降低 ${Math.round(stormReduction * 100)}%`">
          <span class="effect-icon">🌤️</span>
          <span class="effect-text">-{{ Math.round(stormReduction * 100) }}%</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue'

const props = defineProps({
  heat: {
    type: Number,
    default: 0
  },
  canvasSize: {
    type: Number,
    default: 200
  },
  buildings: {
    type: Object,
    default: () => ({ windWall: 0, storageCellar: 0, watchtower: 0 })
  },
  isDay: {
    type: Boolean,
    default: true
  },
  warmReduction: {
    type: Number,
    default: 0
  },
  foodBonus: {
    type: Number,
    default: 0
  },
  stormReduction: {
    type: Number,
    default: 0
  }
})

const buildingSlots = computed(() => [
  {
    id: 'windWall',
    buildingId: 'windWall',
    name: '防风墙',
    icon: '🧱',
    position: 'left',
    style: { top: '38%', left: '-15px' }
  },
  {
    id: 'storageCellar',
    buildingId: 'storageCellar',
    name: '储物地窖',
    icon: '🏚️',
    position: 'right',
    style: { top: '38%', right: '-15px' }
  },
  {
    id: 'watchtower',
    buildingId: 'watchtower',
    name: '瞭望塔',
    icon: '🗼',
    position: 'top',
    style: { top: '-8px', left: '50%', transform: 'translateX(-50%)' }
  }
])

const hasAnyBuilding = computed(() => {
  return props.buildings.windWall > 0 || props.buildings.storageCellar > 0 || props.buildings.watchtower > 0
})

const canvasRef = ref(null)
let animationId = null
let particles = []

class Particle {
  constructor(x, y, size, speed, color) {
    this.x = x
    this.y = y
    this.size = size
    this.speed = speed
    this.color = color
    this.alpha = 1
    this.wobble = Math.random() * Math.PI * 2
    this.wobbleSpeed = 0.05 + Math.random() * 0.05
  }

  update() {
    this.y -= this.speed
    this.wobble += this.wobbleSpeed
    this.x += Math.sin(this.wobble) * 0.5
    this.alpha -= 0.015
    this.size *= 0.98
  }

  draw(ctx) {
    ctx.save()
    ctx.globalAlpha = this.alpha
    ctx.fillStyle = this.color
    ctx.beginPath()
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2)
    ctx.fill()
    ctx.restore()
  }
}

function getFlameColors(intensity) {
  if (intensity > 70) {
    return ['#fff7e6', '#ffd700', '#ff8c00', '#ff4500', '#dc143c']
  } else if (intensity > 40) {
    return ['#ffd700', '#ff8c00', '#ff4500', '#dc143c', '#8b0000']
  } else if (intensity > 10) {
    return ['#ff8c00', '#ff4500', '#dc143c', '#8b0000', '#4a0000']
  } else {
    return ['#8b0000', '#4a0000', '#2d0000', '#1a0000']
  }
}

function createParticle(centerX, baseY, intensity) {
  const colors = getFlameColors(intensity)
  const color = colors[Math.floor(Math.random() * colors.length)]
  const size = 2 + Math.random() * (intensity / 15)
  const speed = 1 + Math.random() * (intensity / 25)
  const x = centerX + (Math.random() - 0.5) * (intensity / 2.5)
  const y = baseY - Math.random() * 10
  
  return new Particle(x, y, size, speed, color)
}

function drawLogs(ctx, centerX, baseY) {
  ctx.fillStyle = '#4a2c0a'
  ctx.strokeStyle = '#2d1a05'
  ctx.lineWidth = 2
  
  ctx.save()
  ctx.translate(centerX - 30, baseY + 10)
  ctx.rotate(-0.3)
  ctx.fillRect(0, 0, 60, 12)
  ctx.strokeRect(0, 0, 60, 12)
  ctx.restore()
  
  ctx.save()
  ctx.translate(centerX - 30, baseY + 15)
  ctx.rotate(0.3)
  ctx.fillRect(0, 0, 60, 12)
  ctx.strokeRect(0, 0, 60, 12)
  ctx.restore()
  
  ctx.fillStyle = '#1a0a00'
  for (let i = 0; i < 5; i++) {
    const emberX = centerX - 20 + i * 10
    const emberY = baseY + 5 + Math.random() * 10
    ctx.beginPath()
    ctx.arc(emberX, emberY, 2 + Math.random() * 2, 0, Math.PI * 2)
    ctx.fill()
  }
}

function drawFlameBase(ctx, centerX, baseY, intensity) {
  if (intensity <= 0) return
  
  const gradient = ctx.createRadialGradient(
    centerX, baseY, 0,
    centerX, baseY, intensity / 1.5
  )
  gradient.addColorStop(0, 'rgba(255, 200, 50, 0.4)')
  gradient.addColorStop(0.5, 'rgba(255, 100, 0, 0.2)')
  gradient.addColorStop(1, 'rgba(255, 0, 0, 0)')
  
  ctx.fillStyle = gradient
  ctx.beginPath()
  ctx.arc(centerX, baseY, intensity / 1.5, 0, Math.PI * 2)
  ctx.fill()
}

function animate() {
  const canvas = canvasRef.value
  if (!canvas) return
  
  const ctx = canvas.getContext('2d')
  const centerX = canvas.width / 2
  const baseY = canvas.height - 50
  const intensity = props.heat
  
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  
  drawFlameBase(ctx, centerX, baseY, intensity)
  drawLogs(ctx, centerX, baseY)
  
  if (intensity > 0) {
    const particleCount = Math.floor(intensity / 5) + 3
    for (let i = 0; i < particleCount; i++) {
      if (Math.random() > 0.5) {
        particles.push(createParticle(centerX, baseY, intensity))
      }
    }
  }
  
  particles = particles.filter(p => p.alpha > 0 && p.y > 0)
  particles.forEach(p => {
    p.update()
    p.draw(ctx)
  })
  
  if (intensity > 0) {
    const glowIntensity = intensity / 100
    const glowGradient = ctx.createRadialGradient(
      centerX, baseY - intensity / 3, 0,
      centerX, baseY - intensity / 3, intensity
    )
    glowGradient.addColorStop(0, `rgba(255, 200, 100, ${0.3 * glowIntensity})`)
    glowGradient.addColorStop(1, 'rgba(255, 100, 50, 0)')
    
    ctx.globalCompositeOperation = 'lighter'
    ctx.fillStyle = glowGradient
    ctx.beginPath()
    ctx.arc(centerX, baseY - intensity / 3, intensity, 0, Math.PI * 2)
    ctx.fill()
    ctx.globalCompositeOperation = 'source-over'
  }
  
  animationId = requestAnimationFrame(animate)
}

watch(() => props.heat, (newVal, oldVal) => {
  if (newVal > oldVal && newVal > 20) {
    for (let i = 0; i < 10; i++) {
      setTimeout(() => {
        const canvas = canvasRef.value
        if (canvas) {
          const centerX = canvas.width / 2
          const baseY = canvas.height - 50
          particles.push(createParticle(centerX, baseY, newVal))
        }
      }, i * 50)
    }
  }
})

onMounted(() => {
  animate()
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
})
</script>

<style scoped>
.campfire-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.campfire-scene {
  position: relative;
}

canvas {
  border-radius: 50%;
  background: radial-gradient(circle, #1a1a2e 0%, #0f0f1a 100%);
  box-shadow: 0 0 30px rgba(255, 100, 50, 0.3);
}

.buildings-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.building-slot {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  transition: all 0.3s ease;
}

.building-slot.occupied {
  z-index: 5;
}

.building-display {
  position: relative;
  background: rgba(0, 0, 0, 0.6);
  border-radius: 12px;
  padding: 6px 8px;
  border: 2px solid rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
  animation: buildingAppear 0.5s ease;
}

@keyframes buildingAppear {
  0% { transform: scale(0); opacity: 0; }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); opacity: 1; }
}

.building-icon-lg {
  font-size: 28px;
}

.building-badge {
  position: absolute;
  top: -6px;
  right: -6px;
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: white;
  font-size: 11px;
  font-weight: bold;
  padding: 2px 6px;
  border-radius: 10px;
  min-width: 22px;
  text-align: center;
  border: 2px solid rgba(255, 255, 255, 0.8);
}

.slot-empty {
  background: rgba(255, 255, 255, 0.1);
  border: 2px dashed rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  padding: 6px 12px;
  opacity: 0.6;
}

.slot-placeholder {
  font-size: 24px;
  color: rgba(255, 255, 255, 0.5);
  font-weight: bold;
}

.slot-label {
  font-size: 10px;
  color: rgba(255, 255, 255, 0.85);
  background: rgba(0, 0, 0, 0.5);
  padding: 2px 6px;
  border-radius: 6px;
  white-space: nowrap;
  font-weight: 500;
}

.slot-warm.occupied .building-display {
  border-color: rgba(255, 150, 100, 0.6);
  box-shadow: 0 0 12px rgba(255, 150, 100, 0.4);
}

.slot-food.occupied .building-display {
  border-color: rgba(100, 200, 100, 0.6);
  box-shadow: 0 0 12px rgba(100, 200, 100, 0.4);
}

.slot-event.occupied .building-display {
  border-color: rgba(100, 150, 255, 0.6);
  box-shadow: 0 0 12px rgba(100, 150, 255, 0.4);
}

.pulse-warm .building-display {
  animation: pulseWarm 2s ease-in-out infinite;
}

@keyframes pulseWarm {
  0%, 100% { box-shadow: 0 0 12px rgba(255, 150, 100, 0.4); }
  50% { box-shadow: 0 0 24px rgba(255, 150, 100, 0.8); }
}

.pulse-food .building-display {
  animation: pulseFood 3s ease-in-out infinite;
}

@keyframes pulseFood {
  0%, 100% { box-shadow: 0 0 12px rgba(100, 200, 100, 0.4); }
  50% { box-shadow: 0 0 24px rgba(100, 200, 100, 0.8); }
}

.pulse-event .building-display {
  animation: pulseEvent 4s ease-in-out infinite;
}

@keyframes pulseEvent {
  0%, 100% { box-shadow: 0 0 12px rgba(100, 150, 255, 0.4); }
  50% { box-shadow: 0 0 24px rgba(100, 150, 255, 0.8); }
}

.effect-indicators {
  position: absolute;
  bottom: -32px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  justify-content: center;
  width: 140%;
}

.effect-badge {
  display: flex;
  align-items: center;
  gap: 3px;
  padding: 3px 8px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: bold;
  backdrop-filter: blur(6px);
  animation: badgePop 0.5s ease;
}

@keyframes badgePop {
  0% { transform: scale(0); }
  70% { transform: scale(1.15); }
  100% { transform: scale(1); }
}

.effect-icon {
  font-size: 14px;
}

.warm-effect {
  background: rgba(255, 150, 100, 0.85);
  color: white;
  border: 1px solid rgba(255, 200, 150, 0.6);
}

.food-effect {
  background: rgba(100, 200, 100, 0.85);
  color: white;
  border: 1px solid rgba(150, 230, 150, 0.6);
}

.event-effect {
  background: rgba(100, 150, 255, 0.85);
  color: white;
  border: 1px solid rgba(150, 180, 255, 0.6);
}
</style>
