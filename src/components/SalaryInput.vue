<template>
  <div class="salary-input">
    <label for="salary">月薪金額</label>
    <div class="input-group">
      <span class="currency">$</span>
      <input
        id="salary"
        type="number"
        :value="modelValue"
        @input="$emit('update:modelValue', Number($event.target.value))"
        placeholder="輸入你的月薪"
        min="0"
      />
      <span class="unit">元</span>
    </div>
  </div>
</template>

<script setup>
defineProps({
  modelValue: {
    type: Number,
    default: 0
  }
})

defineEmits(['update:modelValue'])
</script>

<style scoped>
.salary-input {
  margin-bottom: 1.5rem;
}

.salary-input label {
  display: block;
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-bottom: 0.5rem;
}

.input-group {
  display: flex;
  align-items: center;
  background: var(--bg-input);
  border-radius: 12px;
  padding: 0.75rem 1rem;
  border: 2px solid var(--border-color);
  transition: border-color 0.2s;
}

.input-group:focus-within {
  border-color: var(--primary);
}

.currency {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--primary);
  margin-right: 0.5rem;
}

.input-group input {
  flex: 1;
  border: none;
  background: transparent;
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-primary);
  outline: none;
  width: 100%;
}

.input-group input::placeholder {
  color: var(--text-muted);
  font-weight: 400;
}

.unit {
  color: var(--text-secondary);
  font-size: 1rem;
  margin-left: 0.5rem;
}
</style>
