<template>
  <div class="backup-controls">
    <button class="backup-btn export" @click="exportData" title="匯出本月備份">
      <span class="icon">📥</span>
      <span class="label">匯出</span>
    </button>
    <button class="backup-btn export-all" @click="exportAllData" title="匯出完整歷史">
      <span class="icon">📦</span>
      <span class="label">全部</span>
    </button>
    <label class="backup-btn import" title="匯入備份">
      <span class="icon">📤</span>
      <span class="label">匯入</span>
      <input type="file" accept=".json" @change="importData" hidden />
    </label>
    <button class="backup-btn link" @click="copyLink" title="複製分享連結">
      <span class="icon">🔗</span>
      <span class="label">{{ copied ? '已複製!' : '連結' }}</span>
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  salary: { type: Number, required: true },
  categories: { type: Array, required: true },
  expenses: { type: Array, default: () => [] },
  allData: { type: Object, default: () => ({}) }
})

const emit = defineEmits(['import'])

const copied = ref(false)

// 匯出當月為JSON檔案
const exportData = () => {
  const data = {
    salary: props.salary,
    categories: props.categories,
    expenses: props.expenses,
    exportedAt: new Date().toISOString()
  }
  downloadJson(data, 'salary-backup')
}

// 匯出所有月份
const exportAllData = () => {
  const data = {
    salary: props.salary,
    allData: props.allData,
    exportedAt: new Date().toISOString()
  }
  downloadJson(data, 'salary-full-backup')
}

const downloadJson = (data, prefix) => {
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `${prefix}-${new Date().toISOString().slice(0, 10)}.json`
  a.click()
  URL.revokeObjectURL(url)
}

// 匯入JSON檔案
const importData = (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result)
      if (data.salary && (data.categories || data.allData)) {
        emit('import', data)
        alert('匯入成功！')
      } else {
        alert('無效的備份檔案')
      }
    } catch {
      alert('檔案格式錯誤')
    }
  }
  reader.readAsText(file)
  event.target.value = ''
}

// 複製URL hash連結（僅當月）
const copyLink = () => {
  const data = {
    s: props.salary,
    c: props.categories.map(c => ({
      n: c.name,
      r: c.ratio,
      p: c.spent
    })),
    e: props.expenses
  }
  const hash = encodeURIComponent(JSON.stringify(data))
  const url = `${window.location.origin}${window.location.pathname}#${hash}`
  
  navigator.clipboard.writeText(url).then(() => {
    copied.value = true
    setTimeout(() => { copied.value = false }, 2000)
  })
}
</script>

<style scoped>
.backup-controls {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
  flex-wrap: wrap;
}

.backup-btn {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.5rem 0.9rem;
  border: none;
  border-radius: 10px;
  background: var(--clay-bg);
  box-shadow: var(--clay-shadow-out);
  color: var(--text-secondary);
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s;
}

.backup-btn:hover {
  color: var(--primary);
  transform: translateY(-1px);
}

.backup-btn:active {
  box-shadow: var(--clay-shadow-pressed);
  transform: translateY(0);
}

.backup-btn .icon {
  font-size: 1rem;
}

.backup-btn.link .label {
  min-width: 40px;
}

@media (max-width: 500px) {
  .backup-btn .label {
    display: none;
  }
  .backup-btn {
    padding: 0.6rem;
  }
}
</style>
