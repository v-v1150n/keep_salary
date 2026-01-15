<template>
  <div class="app">
    <header class="app-header">
      <h1>💰 薪資管理</h1>
      <p class="subtitle">輕鬆分配預算，掌握每一分錢</p>
      <div class="top-controls">
        <BackupControls 
          :salary="salary" 
          :categories="categories"
          :expenses="expenses"
          @import="handleImport" 
        />
      </div>
    </header>

    <section class="settings-section clay-card">
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

    <section class="history-section">
      <ExpenseList
        :expenses="expenses"
        :categories="categories"
        @delete="handleDeleteExpense"
        @clear-all="handleClearExpenses"
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
import { computed, watch, onMounted } from 'vue'
import { useStorage } from './composables/useStorage'
import SalaryInput from './components/SalaryInput.vue'
import RatioSelector from './components/RatioSelector.vue'
import CategoryCard from './components/CategoryCard.vue'
import ExpenseForm from './components/ExpenseForm.vue'
import ExpenseList from './components/ExpenseList.vue'
import BackupControls from './components/BackupControls.vue'

// 使用 LocalStorage 持久化
const salary = useStorage('salary-manager-salary', 50000)
const categories = useStorage('salary-manager-categories', [
  { id: 1, name: '生活開銷', ratio: 6, budget: 0, spent: 0 },
  { id: 2, name: '儲蓄', ratio: 3, budget: 0, spent: 0 },
  { id: 3, name: '娛樂', ratio: 1, budget: 0, spent: 0 }
])
const expenses = useStorage('salary-manager-expenses', [])

// 初始化時檢查 URL Hash
onMounted(() => {
  const hash = window.location.hash
  if (hash.length > 1) {
    try {
      const json = decodeURIComponent(hash.slice(1))
      const data = JSON.parse(json)
      
      if (typeof data.s === 'number' && Array.isArray(data.c)) {
        if (confirm('偵測到分享連結資料，是否載入？（目前的資料將被覆蓋）')) {
          salary.value = data.s
          categories.value = data.c.map((c, index) => ({
            id: index + 1,
            name: c.n,
            ratio: c.r,
            budget: 0,
            spent: c.p || 0
          }))
          if (data.e) {
            expenses.value = data.e
          }
          history.replaceState(null, '', ' ')
        }
      }
    } catch (e) {
      console.error('無效的分享連結', e)
    }
  }
})

// 處理 JSON 匯入
const handleImport = (data) => {
  if (confirm('確定要載入備份檔案嗎？會覆蓋目前的設定。')) {
    salary.value = data.salary
    categories.value = data.categories
    if (data.expenses) {
      expenses.value = data.expenses
    }
  }
}

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
const handleAddExpense = ({ categoryId, amount, note }) => {
  const index = categories.value.findIndex(c => c.id === categoryId)
  if (index !== -1) {
    categories.value[index].spent += amount
    // 記錄到 expenses
    expenses.value.push({
      id: Date.now(),
      categoryId,
      amount,
      note,
      date: new Date().toISOString().slice(0, 10)
    })
  }
}

// 刪除單筆支出
const handleDeleteExpense = (expenseId) => {
  const expense = expenses.value.find(e => e.id === expenseId)
  if (expense) {
    // 扣回 spent
    const catIndex = categories.value.findIndex(c => c.id === expense.categoryId)
    if (catIndex !== -1) {
      categories.value[catIndex].spent -= expense.amount
    }
    // 移除紀錄
    expenses.value = expenses.value.filter(e => e.id !== expenseId)
  }
}

// 清除所有支出紀錄
const handleClearExpenses = () => {
  if (confirm('確定要清除所有支出紀錄嗎？')) {
    // 重置所有 spent
    categories.value.forEach(cat => {
      cat.spent = 0
    })
    expenses.value = []
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
    expenses.value = []
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
  position: relative;
}

.app-header h1 {
  font-size: 2.5rem;
  font-weight: 800;
  color: var(--text-primary);
  margin-bottom: 0.5rem;
  letter-spacing: -0.5px;
  text-shadow: 2px 2px 4px rgba(166, 180, 200, 0.4), -2px -2px 4px rgba(255, 255, 255, 0.8);
}

.subtitle {
  color: var(--text-secondary);
  font-size: 1rem;
  font-weight: 500;
}

.top-controls {
  display: flex;
  justify-content: center;
  margin-top: 1rem;
}

.settings-section {
  padding: 2rem;
}

.categories-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}

.expense-section {
  margin-top: 0.5rem;
}

.history-section {
  margin-top: 0.5rem;
}

.app-footer {
  display: flex;
  justify-content: center;
  padding: 1rem 0;
  margin-top: 0.5rem;
}

.reset-btn {
  padding: 0.8rem 1.5rem;
  color: var(--text-secondary);
  background: transparent;
  border: none;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: var(--radius-lg);
}

.reset-btn:hover {
  background: rgba(252, 129, 129, 0.1);
  color: var(--danger);
}

@media (max-width: 480px) {
  .app-header h1 {
    font-size: 2rem;
  }
  
  .settings-section {
    padding: 1.5rem;
  }
}
</style>
