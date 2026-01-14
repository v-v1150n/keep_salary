<template>
  <div class="backup-controls">
    <button class="backup-btn export" @click="exportData" title="匯出備份">
      <span class="icon">📥</span>
      <span class="label">匯出</span>
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
  categories: { type: Array, required: true }
})

const emit = defineEmits(['import'])

const copied = ref(false)

// 匯出為JSON檔案
const exportData = () => {
  const data = {
    salary: props.salary,
    categories: props.categories,
    exportedAt: new Date().toISOString()
  }
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `salary-backup-${new Date().toISOString().slice(0, 10)}.json`
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
      if (data.salary && data.categories) {
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
  event.target.value = '' // 重設以便再次選擇同檔案
}

// 複製URL hash連結
const copyLink = () => {
  const data = {
    s: props.salary,
    c: props.categories.map(c => ({
      n: c.name,
      r: c.ratio,
      p: c.spent
    }))
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
  background: var(--bg-card);
  color: var(--text-secondary);
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s;
  border: 1px solid var(--border-color);
}

.backup-btn:hover {
  background: var(--primary-light);
  color: var(--primary);
  border-color: var(--primary);
}

.backup-btn .icon {
  font-size: 1rem;
}

.backup-btn.link .label {
  min-width: 40px;
}

@media (max-width: 480px) {
  .backup-btn .label {
    display: none;
  }
  .backup-btn {
    padding: 0.6rem;
  }
}
</style>
