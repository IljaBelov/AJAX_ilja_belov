<script setup>
import { ref, watch, onMounted } from 'vue'
import PaginationBar from './components/PaginationBar.vue'

const characters = ref([])
const currentPage = ref(Number(new URLSearchParams(location.search).get('page')) || 1)
const totalPages = ref(1)
const loading = ref(false)
const visible = ref(true)

async function fetchCharacters(page) {
  loading.value = true
  visible.value = false
  const res = await fetch(`https://rickandmortyapi.com/api/character?page=${page}`)
  const data = await res.json()
  characters.value = data.results
  totalPages.value = data.info.pages
  loading.value = false
  setTimeout(() => (visible.value = true), 50)
  window.history.replaceState({}, '', `?page=${page}`)
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function onPageChange(page) {
  currentPage.value = page
}

watch(currentPage, (page) => fetchCharacters(page))
onMounted(() => fetchCharacters(currentPage.value))
</script>

<template>
  <div class="app">
    <h1>Rick & Morty Characters</h1>

    <PaginationBar
      :currentPage="currentPage"
      :totalPages="totalPages"
      @page-change="onPageChange"
    />

    <div v-if="loading" class="loading">Loading...</div>

    <Transition name="fade">
      <div v-if="!loading && visible" class="grid">
        <div v-for="char in characters" :key="char.id" class="card">
          <img :src="char.image" :alt="char.name" />
          <div class="info">
            <h3>{{ char.name }}</h3>
            <span :class="['status', char.status.toLowerCase()]">{{ char.status }}</span>
            <p>{{ char.species }}</p>
          </div>
        </div>
      </div>
    </Transition>

    <PaginationBar
      :currentPage="currentPage"
      :totalPages="totalPages"
      @page-change="onPageChange"
    />
  </div>
</template>

<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body { background: #0d0d1a; color: #fff; font-family: sans-serif; }
.app { max-width: 1200px; margin: 0 auto; padding: 24px; }
h1 { text-align: center; margin-bottom: 8px; color: #00b4d8; font-size: 2rem; }
.loading { text-align: center; padding: 40px; font-size: 1.2rem; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 16px; }
.card { background: #1a1a2e; border-radius: 12px; overflow: hidden; }
.card img { width: 100%; display: block; }
.info { padding: 12px; }
.info h3 { font-size: 1rem; margin-bottom: 6px; }
.info p { font-size: 0.85rem; color: #aaa; margin-top: 4px; }
.status { font-size: 0.8rem; padding: 2px 8px; border-radius: 12px; }
.status.alive { background: #2d6a4f; color: #95d5b2; }
.status.dead { background: #6b2737; color: #ffb3b3; }
.status.unknown { background: #444; color: #ccc; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>
