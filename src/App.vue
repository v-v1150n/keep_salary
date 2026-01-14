<template>
  <div class="app">
    <header class="app-header">
      <h1>💰 薪資管理</h1>
      <p class="subtitle">輕鬆分配預算，掌握每一分錢</p>
    </header>

    <section class="settings-section">
      <SalaryInput v-model="salary" />
      <RatioSelector v-model="categories" />
    </section>

    <section class="categories-grid">
      <CategoryCard
        v-for="category in categoriesWithBudget"
        :key="category.id"
        :category="category"
      />
    </section>

    <section class="expense-section">
      <ExpenseForm
        :categories="categories"
        @add-expense="handleAddExpense"
      />
    </section>

    <footer class="app-footer">
      <button class="reset-btn" @click="resetAll">
        🔄 重置全部
      </button>
    </footer>
  </div>
</template>

<script setup>
import { computed, watch } from 'vue'
import { useStorage } from './composables/useStorage'
import SalaryInput from './components/SalaryInput.vue'
import RatioSelector from './components/RatioSelector.vue'
import CategoryCard from './components/CategoryCard.vue'
import ExpenseForm from './components/ExpenseForm.vue'

// 使用 LocalStorage 持久化
const salary = useStorage('salary-manager-salary', 50000)
const categories = useStorage('salary-manager-categories', [
  { id: 1, name: '生活開銷', ratio: 6, budget: 0, spent: 0 },
  { id: 2, name: '儲蓄', ratio: 3, budget: 0, spent: 0 },
  { id: 3, name: '娛樂', ratio: 1, budget: 0, spent: 0 }
])

// 計算每個類別的預算
const categoriesWithBudget = computed(() => {
  const totalRatio = categories.value.reduce((sum, c) => sum + c.ratio, 0)
  if (totalRatio === 0) return categories.value
  
  return categories.value.map(category => ({
    ...category,
    budget: Math.round((category.ratio / totalRatio) * salary.value)
  }))
})

// 當預算計算後，同步更新到 categories
watch(categoriesWithBudget, (newCats) => {
  newCats.forEach((cat, index) => {
    if (categories.value[index]) {
      categories.value[index].budget = cat.budget
    }
  })
}, { deep: true })

// 處理新增支出
const handleAddExpense = ({ categoryId, amount }) => {
  const index = categories.value.findIndex(c => c.id === categoryId)
  if (index !== -1) {
    categories.value[index].spent += amount
  }
}

// 重置所有資料
const resetAll = () => {
  if (confirm('確定要重置所有資料嗎？')) {
    salary.value = 50000
    categories.value = [
      { id: 1, name: '生活開銷', ratio: 6, budget: 0, spent: 0 },
      { id: 2, name: '儲蓄', ratio: 3, budget: 0, spent: 0 },
      { id: 3, name: '娛樂', ratio: 1, budget: 0, spent: 0 }
    ]
  }
}
</script>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.app-header {
  text-align: center;
  padding: 1rem 0;
}

.app-header h1 {
  font-size: 2rem;
  font-weight: 700;
  background: linear-gradient(135deg, var(--primary), #a855f7);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: var(--text-secondary);
  font-size: 0.95rem;
}

.settings-section {
  background: var(--bg-card);
  border-radius: 20px;
  padding: 1.5rem;
  border: 1px solid var(--border-color);
}

.categories-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
}

.expense-section {
  margin-top: 0.5rem;
}

.app-footer {
  display: flex;
  justify-content: center;
  padding: 1rem 0;
}

.reset-btn {
  padding: 0.6rem 1.25rem;
  background: transparent;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  color: var(--text-secondary);
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}

.reset-btn:hover {
  border-color: var(--danger);
  color: var(--danger);
}

@media (max-width: 480px) {
  .app-header h1 {
    font-size: 1.5rem;
  }
  
  .categories-grid {
    grid-template-columns: 1fr;
  }
}
</style>
