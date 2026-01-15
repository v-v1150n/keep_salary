<template>
  <div class="app">
    <header class="app-header">
      <h1>💰 薪資管理</h1>
      <p class="subtitle">輕鬆分配預算，掌握每一分錢</p>
      <div class="top-controls">
        <BackupControls 
          :salary="salary" 
          :categories="currentCategories"
          :expenses="currentExpenses"
          :allData="allMonthlyData"
          @import="handleImport" 
        />
      </div>
    </header>

    <section class="month-section">
      <MonthSelector v-model="currentMonthKey" />
    </section>

    <section class="settings-section clay-card">
      <SalaryInput v-model="salary" />
      <RatioSelector v-model="currentCategories" />
    </section>

    <section class="summary-section">
      <TotalSummary
        :salary="salary"
        :categories="categoriesWithBudget"
      />
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
        :categories="currentCategories"
        @add-expense="handleAddExpense"
      />
    </section>

    <section class="history-section">
      <ExpenseList
        :expenses="currentExpenses"
        :categories="currentCategories"
        @delete="handleDeleteExpense"
        @clear-all="handleClearExpenses"
      />
    </section>

    <footer class="app-footer">
      <button class="reset-btn" @click="resetCurrentMonth">
        🔄 重置本月
      </button>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { useStorage } from './composables/useStorage'
import SalaryInput from './components/SalaryInput.vue'
import RatioSelector from './components/RatioSelector.vue'
import CategoryCard from './components/CategoryCard.vue'
import ExpenseForm from './components/ExpenseForm.vue'
import ExpenseList from './components/ExpenseList.vue'
import TotalSummary from './components/TotalSummary.vue'
import BackupControls from './components/BackupControls.vue'
import MonthSelector from './components/MonthSelector.vue'

// 取得當前月份 key
const getCurrentMonthKey = () => {
  const now = new Date()
  return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}`
}

// 當前選擇的月份
const currentMonthKey = ref(getCurrentMonthKey())

// 全域設定（跨月份共用）
const salary = useStorage('salary-manager-salary', 50000)

// 所有月份資料
const allMonthlyData = useStorage('salary-manager-monthly', {})

// 預設分類模板
const defaultCategories = [
  { id: 1, name: '生活開銷', ratio: 6, budget: 0, spent: 0 },
  { id: 2, name: '儲蓄', ratio: 3, budget: 0, spent: 0 },
  { id: 3, name: '娛樂', ratio: 1, budget: 0, spent: 0 }
]

// 確保當前月份資料存在
const ensureMonthData = (monthKey) => {
  if (!allMonthlyData.value[monthKey]) {
    // 複製上個月的分類設定（但重置 spent）
    const prevMonth = getPrevMonthKey(monthKey)
    const prevData = allMonthlyData.value[prevMonth]
    
    const categories = prevData?.categories 
      ? prevData.categories.map(c => ({ ...c, spent: 0 }))
      : JSON.parse(JSON.stringify(defaultCategories))
    
    allMonthlyData.value[monthKey] = {
      categories,
      expenses: []
    }
  }
}

const getPrevMonthKey = (monthKey) => {
  const [year, month] = monthKey.split('-').map(Number)
  const date = new Date(year, month - 2)
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`
}

// 當前月份的分類
const currentCategories = computed({
  get: () => {
    ensureMonthData(currentMonthKey.value)
    return allMonthlyData.value[currentMonthKey.value].categories
  },
  set: (val) => {
    ensureMonthData(currentMonthKey.value)
    allMonthlyData.value[currentMonthKey.value].categories = val
  }
})

// 當前月份的支出
const currentExpenses = computed({
  get: () => {
    ensureMonthData(currentMonthKey.value)
    return allMonthlyData.value[currentMonthKey.value].expenses
  },
  set: (val) => {
    ensureMonthData(currentMonthKey.value)
    allMonthlyData.value[currentMonthKey.value].expenses = val
  }
})

// 初始化時檢查 URL Hash
onMounted(() => {
  ensureMonthData(currentMonthKey.value)
  
  const hash = window.location.hash
  if (hash.length > 1) {
    try {
      const json = decodeURIComponent(hash.slice(1))
      const data = JSON.parse(json)
      
      if (typeof data.s === 'number' && Array.isArray(data.c)) {
        if (confirm('偵測到分享連結資料，是否載入？（本月資料將被覆蓋）')) {
          salary.value = data.s
          currentCategories.value = data.c.map((c, index) => ({
            id: index + 1,
            name: c.n,
            ratio: c.r,
            budget: 0,
            spent: c.p || 0
          }))
          if (data.e) {
            currentExpenses.value = data.e
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
  if (data.allData) {
    // 完整備份匯入
    if (confirm('確定要載入完整備份嗎？所有月份資料將被覆蓋。')) {
      salary.value = data.salary
      allMonthlyData.value = data.allData
    }
  } else if (confirm('確定要載入備份嗎？本月資料將被覆蓋。')) {
    salary.value = data.salary
    currentCategories.value = data.categories
    if (data.expenses) {
      currentExpenses.value = data.expenses
    }
  }
}

// 計算每個類別的預算
const categoriesWithBudget = computed(() => {
  const cats = currentCategories.value
  const totalRatio = cats.reduce((sum, c) => sum + c.ratio, 0)
  if (totalRatio === 0) return cats
  
  return cats.map(category => ({
    ...category,
    budget: Math.round((category.ratio / totalRatio) * salary.value)
  }))
})

// 同步預算到分類
watch(categoriesWithBudget, (newCats) => {
  newCats.forEach((cat, index) => {
    if (currentCategories.value[index]) {
      currentCategories.value[index].budget = cat.budget
    }
  })
}, { deep: true })

// 處理新增支出
const handleAddExpense = ({ categoryId, amount, note }) => {
  const cats = currentCategories.value
  const index = cats.findIndex(c => c.id === categoryId)
  if (index !== -1) {
    cats[index].spent += amount
    currentExpenses.value.push({
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
  const expenses = currentExpenses.value
  const expense = expenses.find(e => e.id === expenseId)
  if (expense) {
    const cats = currentCategories.value
    const catIndex = cats.findIndex(c => c.id === expense.categoryId)
    if (catIndex !== -1) {
      cats[catIndex].spent -= expense.amount
    }
    currentExpenses.value = expenses.filter(e => e.id !== expenseId)
  }
}

// 清除當月所有支出
const handleClearExpenses = () => {
  if (confirm('確定要清除本月所有支出紀錄嗎？')) {
    currentCategories.value.forEach(cat => {
      cat.spent = 0
    })
    currentExpenses.value = []
  }
}

// 重置當月資料
const resetCurrentMonth = () => {
  if (confirm('確定要重置本月資料嗎？分類設定與支出都會清空。')) {
    allMonthlyData.value[currentMonthKey.value] = {
      categories: JSON.parse(JSON.stringify(defaultCategories)),
      expenses: []
    }
  }
}
</script>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
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

.month-section {
  margin-top: -0.5rem;
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

/* Tablet */
@media (max-width: 768px) {
  .categories-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
  }
}

/* Mobile */
@media (max-width: 500px) {
  .app {
    gap: 1.25rem;
  }
  
  .app-header h1 {
    font-size: 1.75rem;
  }
  
  .subtitle {
    font-size: 0.9rem;
  }
  
  .settings-section {
    padding: 1.25rem;
  }
  
  .categories-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .reset-btn {
    padding: 0.6rem 1rem;
    font-size: 0.9rem;
  }
}
</style>
