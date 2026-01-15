<template>
  <div class="month-selector">
    <button class="nav-btn" @click="prevMonth" :disabled="!canGoPrev">
      ◀
    </button>
    <div class="current-month" @click="showPicker = !showPicker">
      <span class="year">{{ currentYear }}年</span>
      <span class="month">{{ currentMonth }}月</span>
    </div>
    <button class="nav-btn" @click="nextMonth" :disabled="!canGoNext">
      ▶
    </button>
    
    <button 
      v-if="!isCurrentMonth" 
      class="today-btn clay-button"
      @click="goToCurrentMonth"
    >
      回到本月
    </button>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  modelValue: { type: String, required: true } // format: 'YYYY-MM'
})

const emit = defineEmits(['update:modelValue'])

const currentYear = computed(() => parseInt(props.modelValue.split('-')[0]))
const currentMonth = computed(() => parseInt(props.modelValue.split('-')[1]))

const today = new Date()
const todayKey = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}`

const isCurrentMonth = computed(() => props.modelValue === todayKey)

const canGoNext = computed(() => {
  const [year, month] = props.modelValue.split('-').map(Number)
  const current = new Date(year, month - 1)
  const limit = new Date(today.getFullYear(), today.getMonth())
  return current < limit
})

const canGoPrev = computed(() => true) // 可以無限往前

const prevMonth = () => {
  const [year, month] = props.modelValue.split('-').map(Number)
  const date = new Date(year, month - 2) // month - 1 - 1
  const newKey = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`
  emit('update:modelValue', newKey)
}

const nextMonth = () => {
  const [year, month] = props.modelValue.split('-').map(Number)
  const date = new Date(year, month) // month - 1 + 1
  const newKey = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`
  emit('update:modelValue', newKey)
}

const goToCurrentMonth = () => {
  emit('update:modelValue', todayKey)
}
</script>

<style scoped>
.month-selector {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}

.nav-btn {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: none;
  background: var(--clay-bg);
  box-shadow: var(--clay-shadow-out);
  color: var(--text-primary);
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-btn:hover:not(:disabled) {
  color: var(--primary);
  transform: scale(1.05);
}

.nav-btn:active:not(:disabled) {
  box-shadow: var(--clay-shadow-pressed);
  transform: scale(0.95);
}

.nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.current-month {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0.5rem 1.5rem;
  background: var(--clay-bg);
  border-radius: var(--radius-lg);
  box-shadow: var(--clay-shadow-in);
  cursor: pointer;
  min-width: 100px;
}

.current-month .year {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.current-month .month {
  font-size: 1.5rem;
  font-weight: 800;
  color: var(--text-primary);
}

.today-btn {
  padding: 0.5rem 1rem;
  font-size: 0.85rem;
  color: var(--primary);
}

@media (max-width: 500px) {
  .nav-btn {
    width: 36px;
    height: 36px;
  }
  
  .current-month {
    padding: 0.4rem 1rem;
  }
  
  .current-month .month {
    font-size: 1.25rem;
  }
  
  .today-btn {
    width: 100%;
    margin-top: 0.5rem;
  }
}
</style>
