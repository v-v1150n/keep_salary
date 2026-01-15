<template>
  <form class="expense-form clay-card" @submit.prevent="submitExpense">
    <h3>📝 新增支出</h3>
    <div class="form-row">
      <div class="form-group category-group">
        <label for="category">項目</label>
        <div class="select-wrapper">
          <select id="category" v-model="selectedCategory" class="clay-input" required>
            <option value="" disabled>選擇項目</option>
            <option
              v-for="cat in categories"
              :key="cat.id"
              :value="cat.id"
            >
              {{ cat.name }}
            </option>
          </select>
          <span class="chevron">▼</span>
        </div>
      </div>
      
      <div class="form-group amount-group">
        <label for="amount">金額</label>
        <input
          id="amount"
          type="number"
          v-model.number="amount"
          class="clay-input"
          placeholder="0"
          min="1"
          required
        />
      </div>
      
      <div class="form-group note-group">
        <label for="note">費用來源</label>
        <input
          id="note"
          type="text"
          v-model="note"
          class="clay-input"
          placeholder="例：午餐、交通費"
        />
      </div>
      
      <button type="submit" class="clay-button submit-btn" :disabled="!isValid">
        + 記錄
      </button>
    </div>
  </form>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  categories: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['add-expense'])

const selectedCategory = ref('')
const amount = ref(null)
const note = ref('')

const isValid = computed(() => {
  return selectedCategory.value && amount.value > 0
})

const submitExpense = () => {
  if (!isValid.value) return
  
  emit('add-expense', {
    categoryId: selectedCategory.value,
    amount: amount.value,
    note: note.value || '未標註'
  })
  
  amount.value = null
  note.value = ''
}
</script>

<style scoped>
.expense-form {
  padding: 1.5rem;
}

.expense-form h3 {
  margin: 0 0 1.5rem 0;
  font-size: 1.2rem;
  color: var(--text-primary);
  font-weight: 700;
}

.form-row {
  display: flex;
  gap: 1rem;
  align-items: flex-end;
  flex-wrap: wrap;
}

.form-group {
  min-width: 120px;
}

.form-group.category-group {
  flex: 1.5;
}

.form-group.amount-group {
  flex: 1;
}

.form-group.note-group {
  flex: 2;
}

.form-group label {
  display: block;
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-bottom: 0.5rem;
  font-weight: 600;
  padding-left: 0.5rem;
}

.select-wrapper {
  position: relative;
}

.select-wrapper select {
  width: 100%;
  appearance: none;
  cursor: pointer;
}

.chevron {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  color: var(--text-secondary);
  font-size: 0.8rem;
}

.clay-input {
  width: 100%;
  font-size: 1rem;
}

.submit-btn {
  padding: 0.9rem 1.5rem;
  color: var(--primary-dark);
  font-size: 1rem;
  min-width: 100px;
}

.submit-btn:hover:not(:disabled) {
  color: var(--primary);
}

.submit-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  box-shadow: none !important;
  background: rgba(0,0,0,0.05);
}

@media (max-width: 700px) {
  .form-row {
    flex-direction: column;
    align-items: stretch;
  }
  
  .form-group {
    width: 100%;
  }
  
  .submit-btn {
    width: 100%;
    margin-top: 0.5rem;
  }
}
</style>
