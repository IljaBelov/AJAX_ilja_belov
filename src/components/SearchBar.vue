<script setup>
import { ref } from 'vue'

const emit = defineEmits(['search','input-change'])
const query = ref('')

function submit() {
  emit('search', query.value)
}
</script>

<template>
  <div class="search-bar">
    <input
      v-model="query"
      placeholder="Search characters..."
      @keyup.enter="submit"
      @input="$emit('input-change', $event)"
    />
    <button @click="submit">Search</button>
  </div>
</template>

<style scoped>
.search-bar {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-bottom: 20px;
}
input {
  padding: 10px 16px;
  border-radius: 8px;
  border: 1px solid #444;
  background: #1a1a2e;
  color: #fff;
  font-size: 1rem;
  width: 300px;
  outline: none;
}
input:focus { border-color: #00b4d8; }
button {
  padding: 10px 20px;
  background: #00b4d8;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
}
button:hover { background: #0096c7; }
</style>
