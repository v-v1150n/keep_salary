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

    <!-- Collapsible Expense List -->
    <div class="expense-toggle" v-if="categoryExpenses.length > 0" @click="expanded = !expanded">
      <span class="toggle-icon">{{ expanded ? '▼' : '▶' }}</span>
      <span class="toggle-text">{{ expanded ? '收合' : '展開' }}明細 ({{ categoryExpenses.length }} 筆)</span>
    </div>
    
    <transition name="slide">
      <div class="expense-details" v-if="expanded && categoryExpenses.length > 0">
        <div class="expense-item" v-for="expense in sortedExpenses" :key="expense.id">
          <div class="expense-left">
            <span class="expense-date">{{ expense.date }}</span>
            <span class="expense-note">{{ expense.note || '無備註' }}</span>
          </div>
          <span class="expense-amount">-${{ formatNumber(expense.amount) }}</span>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  category: {
    type: Object,
    required: true
  },
  expenses: {
    type: Array,
    default: () => []
  }
})

const expanded = ref(false)

const categoryExpenses = computed(() => {
  return props.expenses.filter(e => e.categoryId === props.category.id)
})

const sortedExpenses = computed(() => {
  return [...categoryExpenses.value].sort((a, b) => new Date(b.date) - new Date(a.date))
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

/* Collapsible Expense List Styles */
.expense-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 1rem;
  padding: 0.6rem 0.8rem;
  background: var(--bg-main);
  border-radius: var(--radius-md);
  box-shadow: var(--clay-shadow-in);
  cursor: pointer;
  transition: all 0.2s;
}

.expense-toggle:hover {
  background: rgba(166, 180, 200, 0.15);
}

.toggle-icon {
  font-size: 0.7rem;
  color: var(--text-secondary);
  transition: transform 0.2s;
}

.toggle-text {
  font-size: 0.85rem;
  color: var(--text-secondary);
  font-weight: 500;
}

.expense-details {
  margin-top: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  max-height: 200px;
  overflow-y: auto;
}

.expense-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.6rem 0.8rem;
  background: var(--bg-main);
  border-radius: var(--radius-sm);
  box-shadow: var(--clay-shadow-in);
}

.expense-left {
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
}

.expense-date {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.expense-note {
  font-size: 0.85rem;
  color: var(--text-primary);
}

.expense-amount {
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--danger);
}

/* Slide transition */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
  overflow: hidden;
}

.slide-enter-from,
.slide-leave-to {
  opacity: 0;
  max-height: 0;
}

.slide-enter-to,
.slide-leave-from {
  opacity: 1;
  max-height: 200px;
}
</style>
