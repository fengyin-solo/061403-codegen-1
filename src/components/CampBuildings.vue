<template>
  <div class="camp-buildings">
    <h3 class="panel-title">🏕️ 营地设施</h3>
    <div v-if="isNight" class="night-warning">
      <span>🌙 夜晚无法建造设施</span>
    </div>
    <div v-else class="day-hint">
      <span>☀️ 白天可围绕火堆规划设施布局</span>
    </div>
    <div class="buildings-list">
      <div 
        v-for="building in buildingList" 
        :key="building.id"
        class="building-card"
        :class="[
          { disabled: isNight || gameOver || !canBuild(building.id) },
          `building-${building.category}`
        ]"
      >
        <div class="building-header">
          <div class="icon-wrapper">
            <span class="building-icon">{{ building.icon }}</span>
            <span class="position-badge">{{ getPositionText(building.id) }}</span>
          </div>
          <div class="building-info">
            <span class="building-name">{{ building.name }}</span>
            <span class="building-count">
              已建造: <strong>{{ buildings[building.id] }}</strong>
              <span v-if="buildings[building.id] > 0" class="current-effect">
                {{ getCurrentEffectText(building) }}
              </span>
            </span>
          </div>
        </div>
        <p class="building-desc">{{ building.description }}</p>
        <div class="building-effect">
          <span class="effect-label">效果:</span>
          <span class="effect-value">{{ building.effectDesc }}</span>
        </div>
        <div class="building-cost">
          <span class="cost-label">消耗:</span>
          <span class="cost-items">
            <span v-if="building.cost.wood" class="cost-item">🪵{{ building.cost.wood }}</span>
            <span v-if="building.cost.hide" class="cost-item">🦊{{ building.cost.hide }}</span>
            <span v-if="building.cost.tools" class="cost-item">🔧{{ building.cost.tools }}</span>
          </span>
        </div>
        <button 
          class="build-btn"
          :class="{ disabled: isNight || gameOver || !canBuild(building.id) }"
          :disabled="isNight || gameOver || !canBuild(building.id)"
          @click="$emit('build', building.id)"
        >
          {{ buildings[building.id] > 0 ? '升级建造' : '建造' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  buildings: { type: Object, required: true },
  BUILDINGS: { type: Object, required: true },
  isNight: { type: Boolean, default: false },
  gameOver: { type: Boolean, default: false },
  wood: { type: Number, default: 0 },
  hide: { type: Number, default: 0 },
  tools: { type: Number, default: 0 }
})

defineEmits(['build'])

const buildingList = computed(() => Object.values(props.BUILDINGS))

const positionMap = {
  windWall: { text: '左侧', icon: '⬅️' },
  storageCellar: { text: '右侧', icon: '➡️' },
  watchtower: { text: '上方', icon: '⬆️' }
}

function getPositionText(buildingId) {
  return positionMap[buildingId]?.icon || '📍'
}

function getCurrentEffectText(building) {
  const count = props.buildings[building.id] || 0
  if (count === 0) return ''
  switch (building.id) {
    case 'windWall':
      return `（节省${Math.round(building.effect.heatConsumptionReduction * 100 * count)}%热量）`
    case 'storageCellar':
      return `（+${building.effect.dailyFoodBonus * count}食物/天）`
    case 'watchtower':
      return `（降低${Math.min(80, building.effect.blizzardChanceReduction * 100 * count)}%暴雪）`
    default:
      return ''
  }
}

function canBuild(buildingId) {
  const building = props.BUILDINGS[buildingId]
  if (!building) return false
  if (building.cost.wood && props.wood < building.cost.wood) return false
  if (building.cost.hide && props.hide < building.cost.hide) return false
  if (building.cost.tools && props.tools < building.cost.tools) return false
  return true
}
</script>

<style scoped>
.camp-buildings {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 20px;
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.2);
}

.panel-title {
  color: white;
  font-size: 18px;
  margin-bottom: 15px;
  text-align: center;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.night-warning {
  background: rgba(50, 50, 100, 0.8);
  border: 1px solid rgba(100, 100, 200, 0.5);
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 15px;
  text-align: center;
  color: #a0c4ff;
  font-size: 14px;
}

.day-hint {
  background: rgba(255, 200, 100, 0.2);
  border: 1px solid rgba(255, 200, 100, 0.5);
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 15px;
  text-align: center;
  color: #ffe4b5;
  font-size: 13px;
  animation: hintPulse 2s ease-in-out infinite;
}

@keyframes hintPulse {
  0%, 100% { background: rgba(255, 200, 100, 0.15); }
  50% { background: rgba(255, 200, 100, 0.3); }
}

.buildings-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.building-card {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0.05));
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  padding: 12px;
  transition: all 0.2s ease;
}

.building-card:hover:not(.disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  border-color: rgba(255, 200, 100, 0.5);
}

.building-card.disabled {
  opacity: 0.6;
}

.building-header {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  margin-bottom: 8px;
}

.icon-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.building-icon {
  font-size: 32px;
}

.position-badge {
  font-size: 12px;
  background: rgba(0, 0, 0, 0.5);
  padding: 1px 4px;
  border-radius: 6px;
}

.building-info {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.building-name {
  color: white;
  font-weight: bold;
  font-size: 15px;
}

.building-count {
  color: rgba(255, 255, 255, 0.7);
  font-size: 12px;
  line-height: 1.5;
}

.building-count strong {
  color: #ffd700;
  font-weight: bold;
}

.current-effect {
  color: #7fff7f;
  font-weight: 500;
}

.building-warm {
  border-color: rgba(255, 150, 100, 0.3);
}

.building-warm:hover:not(.disabled) {
  border-color: rgba(255, 150, 100, 0.6);
  box-shadow: 0 4px 15px rgba(255, 150, 100, 0.2);
}

.building-food {
  border-color: rgba(100, 200, 100, 0.3);
}

.building-food:hover:not(.disabled) {
  border-color: rgba(100, 200, 100, 0.6);
  box-shadow: 0 4px 15px rgba(100, 200, 100, 0.2);
}

.building-event {
  border-color: rgba(100, 150, 255, 0.3);
}

.building-event:hover:not(.disabled) {
  border-color: rgba(100, 150, 255, 0.6);
  box-shadow: 0 4px 15px rgba(100, 150, 255, 0.2);
}

.building-desc {
  color: rgba(255, 255, 255, 0.8);
  font-size: 12px;
  margin: 6px 0;
  line-height: 1.4;
}

.building-effect,
.building-cost {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  margin-top: 4px;
}

.effect-label,
.cost-label {
  color: rgba(255, 255, 255, 0.6);
}

.effect-value {
  color: #7fff7f;
  font-weight: bold;
}

.cost-items {
  display: flex;
  gap: 8px;
}

.cost-item {
  color: rgba(255, 255, 255, 0.9);
}

.build-btn {
  width: 100%;
  margin-top: 10px;
  padding: 8px;
  background: linear-gradient(135deg, #4a90d9, #357abd);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
  font-size: 14px;
}

.build-btn:hover:not(.disabled) {
  transform: translateY(-1px);
  box-shadow: 0 3px 10px rgba(74, 144, 217, 0.5);
}

.build-btn:active:not(.disabled) {
  transform: translateY(0);
}

.build-btn.disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(0.5);
}
</style>
