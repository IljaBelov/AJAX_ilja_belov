<script setup>
import { ref, watch, onMounted } from 'vue'
import PaginationBar from './components/PaginationBar.vue'
import SearchBar from './components/SearchBar.vue'

const characters = ref([])
const currentPage = ref(Number(new URLSearchParams(location.search).get('page')) || 1)
const totalPages = ref(1)
const totalCount = ref(0)
const loading = ref(false)
const visible = ref(true)
const notFound = ref(false)
const searchQuery = ref('')

let abortController = null

function debounce(fn, delay) {
  let timer = null
  return function (...args) {
    clearTimeout(timer)
    timer = setTimeout(() => fn(...args), delay)
  }
}

async function fetchCharacters(page = currentPage.value, query = searchQuery.value) {
  if (abortController) abortController.abort()
  abortController = new AbortController()

  loading.value = true
  visible.value = false
  notFound.value = false

  try {
    let url = `https://rickandmortyapi.com/api/character?page=${page}`
    if (query) url += `&name=${encodeURIComponent(query)}`

    const res = await fetch(url, { signal: abortController.signal })

    if (res.status === 404) {
      characters.value = []
      totalPages.value = 1
      totalCount.value = 0
      notFound.value = true
      loading.value = false
      setTimeout(() => (visible.value = true), 50)
      return
    }

    const data = await res.json()
    characters.value = data.results
    totalPages.value = data.info.pages
    totalCount.value = data.info.count
    loading.value = false
    setTimeout(() => (visible.value = true), 50)
    window.history.replaceState({}, '', `?page=${page}`)
    window.scrollTo({ top: 0, behavior: 'smooth' })
  } catch (e) {
    if (e.name !== 'AbortError') console.error(e)
    loading.value = false
  }
}

// Stage 2 комментарий:
// Questions:
// 1. How many API requests were made while typing "morty"? — 5 requests (m, mo, mor, mort, morty)
// 2. What happens if a slow request from keystroke 2 arrives after keystroke 5? — результаты от "mo" перезапишут результаты "morty" (race condition)
// 3. How does this affect the server and the user experience? — лишняя нагрузка на сервер и неправильные результаты для пользователя

const debouncedFetch = debounce((query) => {
  currentPage.value = 1
  fetchCharacters(1, query)
}, 400)

function onSearch(query) {
  searchQuery.value = query
  currentPage.value = 1
  fetchCharacters(1, query)
}

function onInput(e) {
  searchQuery.value = e.target.value
  debouncedFetch(e.target.value)
}

function onPageChange(page) {
  currentPage.value = page
  fetchCharacters(page, searchQuery.value)
}

onMounted(() => fetchCharacters())
</script>

<template>
  <div class="app">
    <h1>Rick & Morty Characters</h1>

    <SearchBar @search="onSearch" @input-change="onInput" />

    <p v-if="totalCount && !notFound" class="count">Found: {{ totalCount }} characters</p>

    <PaginationBar
      v-if="!notFound"
      :currentPage="currentPage"
      :totalPages="totalPages"
      @page-change="onPageChange"
    />

    <div v-if="loading" class="loading">
      <div class="spinner"></div>
    </div>

    <div v-if="notFound && !loading" class="not-found">No characters found 👽</div>

    <Transition name="fade">
      <div v-if="!loading && visible && !notFound" class="grid">
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
      v-if="!notFound && characters.length"
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
h1 { text-align: center; margin-bottom: 20px; color: #00b4d8; font-size: 2rem; }
.count { text-align: center; color: #aaa; font-size: 0.9rem; margin-bottom: 8px; }
.loading { display: flex; justify-content: center; padding: 60px; }
.spinner {
  width: 40px; height: 40px;
  border: 4px solid #333;
  border-top-color: #00b4d8;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
.not-found { text-align: center; font-size: 1.5rem; padding: 60px; color: #aaa; }
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
