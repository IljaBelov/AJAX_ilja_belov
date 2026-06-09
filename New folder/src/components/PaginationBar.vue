<script setup>
import { computed } from 'vue'

const props = defineProps({
  currentPage: Number,
  totalPages: Number,
})

const emit = defineEmits(['page-change'])

function go(page) {
  if (page < 1 || page > props.totalPages) return
  emit('page-change', page)
}

const pages = computed(() => {
  const total = props.totalPages
  const current = props.currentPage
  if (total <= 7) return Array.from({ length: total }, (_, i) => i + 1)

  if (current <= 4) return [1, 2, 3, 4, 5, '...', total]
  if (current >= total - 3) return [1, '...', total-4, total-3, total-2, total-1, total]
  return [1, '...', current-1, current, current+1, '...', total]
})
</script>

<template>
  <div class="pagination">
    <button :disabled="currentPage === 1" @click="go(currentPage - 1)">← Prev</button>

    <button
      v-for="(page, i) in pages"
      :key="i"
      :class="{ active: page === currentPage, ellipsis: page === '...' }"
      :disabled="page === '...'"
      @click="go(page)"
    >
      {{ page }}
    </button>

    <button :disabled="currentPage === totalPages" @click="go(currentPage + 1)">Next →</button>
  </div>
</template>

<style scoped>
.pagination {
  display: flex;
  gap: 6px;
  justify-content: center;
  flex-wrap: wrap;
  margin: 24px 0;
}
button {
  padding: 8px 14px;
  border: 1px solid #444;
  background: #1a1a2e;
  color: #fff;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}
button:hover:not(:disabled) { background: #00b4d8; }
button.active { background: #00b4d8; font-weight: bold; border-color: #00b4d8; }
button:disabled { opacity: 0.4; cursor: default; }
button.ellipsis { border: none; background: transparent; }
</style>
