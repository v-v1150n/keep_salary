<template>
  <div class="ratio-selector">
    <label>分割模板</label>
    <div class="preset-buttons">
      <button
        v-for="preset in presets"
        :key="preset.name"
        :class="['preset-btn', { active: isActivePreset(preset) }]"
        @click="selectPreset(preset)"
      >
        {{ preset.name }}
      </button>
    </div>
    
    <div v-if="isCustomMode" class="custom-ratios">
      <div
        v-for="(category, index) in modelValue"
        :key="index"
        class="custom-item"
      >
        <input
          type="text"
          :value="category.name"
          @input="updateCategory(index, 'name', $event.target.value)"
          placeholder="項目名稱"
          class="name-input"
        />
        <input
          type="number"
          :value="category.ratio"
          @input="updateCategory(index, 'ratio', Number($event.target.value))"
          placeholder="比例"
          min="0.1"
          step="0.1"
          class="ratio-input"
        />
        <button
          v-if="modelValue.length > 1"
          class="remove-btn"
          @click="removeCategory(index)"
        >
          ✕
        </button>
      </div>
      <button class="add-btn" @click="addCategory">+ 新增項目</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['update:modelValue'])

const presets = [
  { name: '6/3/1', ratios: [6, 3, 1], labels: ['生活開銷', '儲蓄', '娛樂'] },
  { name: '5/3/2', ratios: [5, 3, 2], labels: ['生活', '儲蓄', '投資'] },
  { name: '3.5/3.5/3', ratios: [3.5, 3.5, 3], labels: ['必要', '儲蓄', '彈性'] },
  { name: '自訂', ratios: null, labels: null }
]

const isCustomMode = ref(false)

const isActivePreset = (preset) => {
  if (preset.ratios === null) return isCustomMode.value
  if (isCustomMode.value) return false
  if (props.modelValue.length !== preset.ratios.length) return false
  return preset.ratios.every((r, i) => props.modelValue[i]?.ratio === r)
}

const selectPreset = (preset) => {
  if (preset.ratios === null) {
    isCustomMode.value = true
    return
  }
  isCustomMode.value = false
  const categories = preset.ratios.map((ratio, i) => ({
    id: i + 1,
    name: preset.labels[i],
    ratio,
    budget: 0,
    spent: 0
  }))
  emit('update:modelValue', categories)
}

const updateCategory = (index, field, value) => {
  const updated = [...props.modelValue]
  updated[index] = { ...updated[index], [field]: value }
  emit('update:modelValue', updated)
}

const addCategory = () => {
  const newId = Math.max(...props.modelValue.map(c => c.id), 0) + 1
  emit('update:modelValue', [
    ...props.modelValue,
    { id: newId, name: '新項目', ratio: 1, budget: 0, spent: 0 }
  ])
}

const removeCategory = (index) => {
  const updated = props.modelValue.filter((_, i) => i !== index)
  emit('update:modelValue', updated)
}
</script>

<style scoped>
.ratio-selector {
  margin-bottom: 1.5rem;
}

.ratio-selector > label {
  display: block;
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-bottom: 0.75rem;
}

.preset-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.preset-btn {
  padding: 0.6rem 1rem;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  background: var(--bg-card);
  color: var(--text-primary);
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}

.preset-btn:hover {
  border-color: var(--primary);
}

.preset-btn.active {
  background: var(--primary);
  border-color: var(--primary);
  color: white;
}

.custom-ratios {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.custom-item {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.name-input {
  flex: 2;
  padding: 0.6rem 0.75rem;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  background: var(--bg-input);
  color: var(--text-primary);
  font-size: 0.9rem;
}

.ratio-input {
  flex: 1;
  padding: 0.6rem 0.75rem;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  background: var(--bg-input);
  color: var(--text-primary);
  font-size: 0.9rem;
  text-align: center;
}

.name-input:focus,
.ratio-input:focus {
  outline: none;
  border-color: var(--primary);
}

.remove-btn {
  width: 32px;
  height: 32px;
  border: none;
  border-radius: 8px;
  background: var(--danger);
  color: white;
  cursor: pointer;
  font-size: 0.8rem;
  transition: opacity 0.2s;
}

.remove-btn:hover {
  opacity: 0.85;
}

.add-btn {
  padding: 0.6rem;
  border: 2px dashed var(--border-color);
  border-radius: 8px;
  background: transparent;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s;
}

.add-btn:hover {
  border-color: var(--primary);
  color: var(--primary);
}
</style>
