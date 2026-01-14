<template>
  <form class="expense-form" @submit.prevent="submitExpense">
    <h3>📝 新增支出</h3>
    <div class="form-row">
      <div class="form-group">
        <label for="category">項目</label>
        <select id="category" v-model="selectedCategory" required>
          <option value="" disabled>選擇項目</option>
          <option
            v-for="cat in categories"
            :key="cat.id"
            :value="cat.id"
          >
            {{ cat.name }}
          </option>
        </select>
      </div>
      
      <div class="form-group">
        <label for="amount">金額</label>
        <input
          id="amount"
          type="number"
          v-model.number="amount"
          placeholder="0"
          min="1"
          required
        />
      </div>
      
      <button type="submit" class="submit-btn" :disabled="!isValid">
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

const isValid = computed(() => {
  return selectedCategory.value && amount.value > 0
})

const submitExpense = () => {
  if (!isValid.value) return
  
  emit('add-expense', {
    categoryId: selectedCategory.value,
    amount: amount.value
  })
  
  amount.value = null
}
</script>

<style scoped>
.expense-form {
  background: var(--bg-card);
  border-radius: 16px;
  padding: 1.25rem;
  border: 2px solid var(--border-color);
}

.expense-form h3 {
  margin: 0 0 1rem 0;
  font-size: 1rem;
  color: var(--text-primary);
}

.form-row {
  display: flex;
  gap: 0.75rem;
  align-items: flex-end;
  flex-wrap: wrap;
}

.form-group {
  flex: 1;
  min-width: 120px;
}

.form-group label {
  display: block;
  font-size: 0.8rem;
  color: var(--text-secondary);
  margin-bottom: 0.4rem;
}

.form-group select,
.form-group input {
  width: 100%;
  padding: 0.7rem 0.75rem;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  background: var(--bg-input);
  color: var(--text-primary);
  font-size: 0.95rem;
}

.form-group select:focus,
.form-group input:focus {
  outline: none;
  border-color: var(--primary);
}

.submit-btn {
  padding: 0.7rem 1.25rem;
  background: var(--primary);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}

.submit-btn:hover:not(:disabled) {
  background: var(--primary-dark);
  transform: translateY(-1px);
}

.submit-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
