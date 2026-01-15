<template>
  <div class="total-summary clay-card">
    <div class="summary-header">
      <h2>💵 總覽</h2>
    </div>
    <div class="summary-grid">
      <div class="summary-item">
        <span class="label">月薪</span>
        <span class="value salary">${{ formatNumber(salary) }}</span>
      </div>
      <div class="summary-item">
        <span class="label">已花費</span>
        <span class="value spent">${{ formatNumber(totalSpent) }}</span>
      </div>
      <div class="summary-item highlight">
        <span class="label">剩餘可用</span>
        <span class="value remaining" :class="{ danger: remaining < 0 }">
          ${{ formatNumber(remaining) }}
        </span>
      </div>
    </div>
    <div class="progress-section">
      <div class="progress-outer">
        <div class="progress-inner" :style="{ width: usagePercent + '%' }"></div>
      </div>
      <span class="progress-text">已使用 {{ usagePercent.toFixed(0) }}%</span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  salary: { type: Number, required: true },
  categories: { type: Array, required: true }
})

const totalSpent = computed(() => {
  return props.categories.reduce((sum, c) => sum + (c.spent || 0), 0)
})

const remaining = computed(() => {
  return props.salary - totalSpent.value
})

const usagePercent = computed(() => {
  if (props.salary === 0) return 0
  return Math.min((totalSpent.value / props.salary) * 100, 100)
})

const formatNumber = (num) => {
  return num.toLocaleString('zh-TW')
}
</script>

<style scoped>
.total-summary {
  padding: 1.5rem;
  background: linear-gradient(135deg, var(--clay-bg) 0%, #e8edf5 100%);
}

.summary-header h2 {
  margin: 0 0 1.25rem 0;
  font-size: 1.3rem;
  font-weight: 700;
  color: var(--text-primary);
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.summary-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
  padding: 1rem 0.5rem;
  background: var(--bg-main);
  border-radius: var(--radius-lg);
  box-shadow: var(--clay-shadow-in);
}

.summary-item.highlight {
  background: linear-gradient(135deg, rgba(104, 211, 145, 0.15), rgba(104, 211, 145, 0.05));
}

.summary-item .label {
  font-size: 0.85rem;
  color: var(--text-secondary);
  font-weight: 500;
}

.summary-item .value {
  font-size: 1.4rem;
  font-weight: 800;
}

.summary-item .value.salary {
  color: var(--text-primary);
}

.summary-item .value.spent {
  color: var(--primary-dark);
}

.summary-item .value.remaining {
  color: var(--success);
}

.summary-item .value.remaining.danger {
  color: var(--danger);
}

.progress-section {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.progress-outer {
  flex: 1;
  height: 10px;
  background: var(--bg-main);
  border-radius: 8px;
  box-shadow: var(--clay-shadow-in);
  overflow: hidden;
}

.progress-inner {
  height: 100%;
  background: linear-gradient(90deg, var(--primary), var(--primary-dark));
  border-radius: 8px;
  transition: width 0.4s ease;
}

.progress-text {
  font-size: 0.85rem;
  color: var(--text-secondary);
  font-weight: 600;
  white-space: nowrap;
}

@media (max-width: 500px) {
  .summary-grid {
    grid-template-columns: 1fr;
    gap: 0.75rem;
  }
  
  .summary-item {
    flex-direction: row;
    justify-content: space-between;
    padding: 0.75rem 1rem;
  }
  
  .summary-item .value {
    font-size: 1.2rem;
  }
}
</style>
