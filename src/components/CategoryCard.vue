<template>
  <div class="category-card" :class="{ warning: percentage > 80, danger: percentage > 95 }">
    <div class="card-header">
      <span class="icon">{{ getIcon(category.name) }}</span>
      <h3>{{ category.name }}</h3>
      <span class="ratio-badge">{{ category.ratio }}</span>
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

    <div class="progress-container">
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: percentage + '%' }"></div>
      </div>
      <span class="percentage">{{ percentage.toFixed(0) }}%</span>
    </div>

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
  background: var(--bg-card);
  border-radius: 16px;
  padding: 1.25rem;
  border: 2px solid var(--border-color);
  transition: all 0.3s;
}

.category-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.category-card.warning {
  border-color: var(--warning);
}

.category-card.danger {
  border-color: var(--danger);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.icon {
  font-size: 1.5rem;
}

.card-header h3 {
  flex: 1;
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--text-primary);
}

.ratio-badge {
  background: var(--primary-light);
  color: var(--primary);
  padding: 0.25rem 0.5rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
}

.budget-info {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  margin-bottom: 1rem;
}

.budget-row {
  display: flex;
  justify-content: space-between;
  font-size: 0.9rem;
}

.budget-row .label {
  color: var(--text-secondary);
}

.budget-row .value {
  color: var(--text-primary);
  font-weight: 500;
}

.budget-row .value.spent {
  color: var(--primary);
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.progress-bar {
  flex: 1;
  height: 8px;
  background: var(--bg-input);
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--primary), var(--primary-dark));
  border-radius: 4px;
  transition: width 0.3s ease;
}

.category-card.warning .progress-fill {
  background: linear-gradient(90deg, var(--warning), #e67e00);
}

.category-card.danger .progress-fill {
  background: linear-gradient(90deg, var(--danger), #c0392b);
}

.percentage {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-secondary);
  min-width: 36px;
  text-align: right;
}

.remaining {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 0.75rem;
  border-top: 1px solid var(--border-color);
}

.remaining .label {
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.remaining .value {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--success);
}

.remaining .value.negative {
  color: var(--danger);
}
</style>
