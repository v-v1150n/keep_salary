<template>
  <div class="ratio-selector">
    <label>分割模板</label>
    <div class="preset-buttons">
      <button
        v-for="preset in presets"
        :key="preset.name"
        :class="['clay-button preset-btn', { active: isActivePreset(preset) }]"
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
          class="clay-input name-input"
        />
        <input
          type="number"
          :value="category.ratio"
          @input="updateCategory(index, 'ratio', Number($event.target.value))"
          placeholder="權重"
          min="0.1"
          step="0.1"
          class="clay-input ratio-input"
        />
        <button
          v-if="modelValue.length > 1"
          class="delete-btn"
          @click="removeCategory(index)"
        >
          ✕
        </button>
      </div>
      <button class="clay-button add-btn" @click="addCategory">+ 新增項目</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

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
.ratio-selector > label {
  display: block;
  font-size: 0.95rem;
  color: var(--text-secondary);
  margin-bottom: 0.8rem;
  font-weight: 600;
  padding-left: 0.5rem;
}

.preset-buttons {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.preset-btn {
  padding: 0.8rem 1.5rem;
  font-size: 1rem;
}

.preset-btn.active {
  color: var(--primary);
  box-shadow: var(--clay-shadow-pressed);
}

.custom-ratios {
  margin-top: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.custom-item {
  display: flex;
  gap: 0.8rem;
  align-items: center;
}

.name-input {
  flex: 2;
  font-size: 1rem;
}

.ratio-input {
  flex: 1;
  text-align: center;
  font-size: 1rem;
}

.delete-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: var(--clay-bg);
  box-shadow: var(--clay-shadow-out);
  color: var(--danger);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.delete-btn:hover {
  transform: scale(1.1);
  color: #c53030;
}

.delete-btn:active {
  box-shadow: var(--clay-shadow-pressed);
  transform: scale(0.95);
}

.add-btn {
  width: 100%;
  margin-top: 0.5rem;
  padding: 0.8rem;
  color: var(--text-secondary);
}

.add-btn:hover {
  color: var(--primary);
}
</style>
