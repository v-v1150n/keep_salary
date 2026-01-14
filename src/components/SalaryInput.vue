<template>
  <div class="salary-input">
    <label for="salary">月薪金額</label>
    <div class="input-wrapper">
      <span class="currency">$</span>
      <input
        id="salary"
        type="number"
        :value="modelValue"
        class="clay-input"
        @input="$emit('update:modelValue', Number($event.target.value))"
        placeholder="輸入你的月薪"
        min="0"
      />
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
  margin-bottom: 2rem;
}

.salary-input label {
  display: block;
  font-size: 0.95rem;
  color: var(--text-secondary);
  margin-bottom: 0.8rem;
  font-weight: 600;
  padding-left: 0.5rem;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.currency {
  position: absolute;
  left: 1.2rem;
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--primary);
  z-index: 1;
}

input.clay-input {
  width: 100%;
  padding: 1rem 1rem 1rem 3rem;
  font-size: 1.8rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: 1px;
}

input.clay-input::placeholder {
  color: #cbd5e0;
  font-weight: 400;
  font-size: 1.2rem;
}
</style>
