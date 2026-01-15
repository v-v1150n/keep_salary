<template>
  <div class="expense-list clay-card" v-if="expenses.length > 0">
    <div class="list-header">
      <h3>📋 支出紀錄</h3>
      <button class="clear-btn" @click="$emit('clear-all')" title="清除全部">
        🗑️
      </button>
    </div>
    <div class="list-content">
      <div 
        v-for="expense in sortedExpenses" 
        :key="expense.id" 
        class="expense-item"
      >
        <div class="expense-info">
          <span class="expense-category">{{ getCategoryName(expense.categoryId) }}</span>
          <span class="expense-note">{{ expense.note }}</span>
        </div>
        <div class="expense-right">
          <span class="expense-amount">-${{ formatNumber(expense.amount) }}</span>
          <button class="delete-btn" @click="$emit('delete', expense.id)">✕</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  expenses: {
    type: Array,
    default: () => []
  },
  categories: {
    type: Array,
    default: () => []
  }
})

defineEmits(['delete', 'clear-all'])

const sortedExpenses = computed(() => {
  return [...props.expenses].reverse() // 最新的在前
})

const getCategoryName = (categoryId) => {
  const cat = props.categories.find(c => c.id === categoryId)
  return cat ? cat.name : '未知'
}

const formatNumber = (num) => {
  return num.toLocaleString('zh-TW')
}
</script>

<style scoped>
.expense-list {
  padding: 1.5rem;
  max-height: 300px;
  display: flex;
  flex-direction: column;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.list-header h3 {
  margin: 0;
  font-size: 1.1rem;
  color: var(--text-primary);
  font-weight: 700;
}

.clear-btn {
  background: transparent;
  border: none;
  font-size: 1rem;
  cursor: pointer;
  opacity: 0.6;
  transition: opacity 0.2s;
}

.clear-btn:hover {
  opacity: 1;
}

.list-content {
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.expense-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  background: var(--bg-main);
  border-radius: var(--radius-md);
  box-shadow: var(--clay-shadow-in);
}

.expense-info {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.expense-category {
  font-size: 0.85rem;
  color: var(--primary-dark);
  font-weight: 600;
}

.expense-note {
  font-size: 0.9rem;
  color: var(--text-primary);
}

.expense-right {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.expense-amount {
  font-size: 1rem;
  font-weight: 700;
  color: var(--danger);
}

.delete-btn {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  border: none;
  background: transparent;
  color: var(--text-secondary);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  transition: all 0.2s;
}

.delete-btn:hover {
  background: rgba(252, 129, 129, 0.2);
  color: var(--danger);
}
</style>
