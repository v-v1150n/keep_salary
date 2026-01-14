<template>
  <div class="category-card clay-card" :class="{ warning: percentage > 80, danger: percentage > 95 }">
    <div class="card-header">
      <div class="icon-bubble">{{ getIcon(category.name) }}</div>
      <h3>{{ category.name }}</h3>
      <span class="ratio-badge">權重 {{ category.ratio }}</span>
    </div>
    
    <div class="budget-info">
      <div class="budget-row">
        <span class="label">預算</span>
        <span class="value">${{ formatNumber(category.budget) }}</span>
      </div>
      <div class="budget-row">
        <span class="label">已用</span>
        <span class="value spent">${{ formatNumber(category.spent) }}</span>
      </div>
    </div>

    <div class="progress-outer">
      <div class="progress-inner" :style="{ width: percentage + '%' }"></div>
    </div>
    <div class="percentage-text">{{ percentage.toFixed(0) }}%</div>

    <div class="remaining">
      <span class="label">剩餘</span>
      <span class="value" :class="{ negative: remaining < 0 }">
        ${{ formatNumber(remaining) }}
      </span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  category: {
    type: Object,
    required: true
  }
})

const remaining = computed(() => props.category.budget - props.category.spent)

const percentage = computed(() => {
  if (props.category.budget === 0) return 0
  return Math.min((props.category.spent / props.category.budget) * 100, 100)
})

const formatNumber = (num) => {
  return num.toLocaleString('zh-TW')
}

const getIcon = (name) => {
  const icons = {
    '生活開銷': '🏠',
    '生活': '🏠',
    '儲蓄': '💎',
    '娛樂': '🎮',
    '投資': '📈',
    '必要': '📋',
    '彈性': '🎯'
  }
  return icons[name] || '💰'
}
</script>

<style scoped>
.category-card {
  padding: 1.5rem;
  position: relative;
  overflow: hidden;
}

.category-card.warning .progress-inner {
  background: var(--warning);
}

.category-card.danger .progress-inner {
  background: var(--danger);
  box-shadow: 0 0 10px var(--danger);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.icon-bubble {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: var(--clay-bg);
  box-shadow: var(--clay-shadow-out);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  color: var(--text-primary);
}

.card-header h3 {
  flex: 1;
  margin: 0;
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--text-primary);
}

.ratio-badge {
  font-size: 0.75rem;
  color: var(--text-secondary);
  font-weight: 600;
  background: rgba(166, 180, 200, 0.2);
  padding: 0.2rem 0.6rem;
  border-radius: 20px;
}

.budget-info {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1.2rem;
}

.budget-row {
  display: flex;
  justify-content: space-between;
  font-size: 0.95rem;
}

.budget-row .label {
  color: var(--text-secondary);
}

.budget-row .value {
  color: var(--text-primary);
  font-weight: 600;
}

.budget-row .value.spent {
  color: var(--primary-dark);
}

.progress-outer {
  height: 12px;
  background: var(--bg-main);
  border-radius: 10px;
  box-shadow: var(--clay-shadow-in);
  margin-bottom: 0.5rem;
  overflow: hidden;
  position: relative;
}

.progress-inner {
  height: 100%;
  background: var(--primary);
  border-radius: 10px;
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.percentage-text {
  text-align: right;
  font-size: 0.8rem;
  font-weight: 700;
  color: var(--text-secondary);
  margin-bottom: 1rem;
}

.remaining {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid rgba(166, 180, 200, 0.3);
}

.remaining .label {
  font-size: 0.9rem;
  color: var(--text-secondary);
  font-weight: 500;
}

.remaining .value {
  font-size: 1.5rem;
  font-weight: 800;
  color: var(--success);
  text-shadow: 1px 1px 2px rgba(255,255,255,0.8);
}

.remaining .value.negative {
  color: var(--danger);
}
</style>
