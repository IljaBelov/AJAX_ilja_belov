<script setup>
import { ref } from 'vue'

const inputText = ref('')
const hashResult = ref('')
const encryptedResult = ref('')
const decryptedResult = ref('')
const error = ref('')
const activeTab = ref('hash')

// --- HASH ---
async function hashText() {
  error.value = ''
  hashResult.value = ''
  if (!inputText.value) return

  try {
    const encoder = new TextEncoder()
    const data = encoder.encode(inputText.value)
    const hashBuffer = await crypto.subtle.digest('SHA-256', data)
    const hashArray = Array.from(new Uint8Array(hashBuffer))
    hashResult.value = hashArray.map(b => b.toString(16).padStart(2, '0')).join('')
  } catch (e) {
    error.value = 'Hashing failed: ' + e.message
  }
}

// --- ENCRYPT / DECRYPT ---
let cryptoKey = null
const encryptInput = ref('')

async function generateKey() {
  cryptoKey = await crypto.subtle.generateKey(
    { name: 'AES-GCM', length: 256 },
    true,
    ['encrypt', 'decrypt']
  )
}

let iv = null

async function encryptText() {
  error.value = ''
  encryptedResult.value = ''
  if (!encryptInput.value) return

  try {
    if (!cryptoKey) await generateKey()
    iv = crypto.getRandomValues(new Uint8Array(12))
    const encoder = new TextEncoder()
    const encoded = encoder.encode(encryptInput.value)
    const encrypted = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, cryptoKey, encoded)
    encryptedResult.value = btoa(String.fromCharCode(...new Uint8Array(encrypted)))
  } catch (e) {
    error.value = 'Encryption failed: ' + e.message
  }
}

async function decryptText() {
  error.value = ''
  decryptedResult.value = ''
  if (!encryptedResult.value || !cryptoKey || !iv) {
    error.value = 'Nothing to decrypt. Encrypt something first.'
    return
  }

  try {
    const encryptedBytes = Uint8Array.from(atob(encryptedResult.value), c => c.charCodeAt(0))
    const decrypted = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, cryptoKey, encryptedBytes)
    decryptedResult.value = new TextDecoder().decode(decrypted)
  } catch (e) {
    error.value = 'Decryption failed: ' + e.message
  }
}

// --- RANDOM ---
const randomResult = ref('')
function generateRandom() {
  const array = new Uint8Array(16)
  crypto.getRandomValues(array)
  randomResult.value = Array.from(array).map(b => b.toString(16).padStart(2, '0')).join('')
}
</script>

<template>
  <div class="app">
    <h1>🔐 Web Crypto API Demo</h1>
    <p class="subtitle">Browser-native cryptography — no libraries needed</p>

    <div class="tabs">
      <button :class="{ active: activeTab === 'hash' }" @click="activeTab = 'hash'">SHA-256 Hash</button>
      <button :class="{ active: activeTab === 'encrypt' }" @click="activeTab = 'encrypt'">AES Encrypt/Decrypt</button>
      <button :class="{ active: activeTab === 'random' }" @click="activeTab = 'random'">Random Values</button>
    </div>

    <!-- HASH -->
    <div v-if="activeTab === 'hash'" class="panel">
      <h2>SHA-256 Hashing</h2>
      <p>Converts any text into a fixed-length fingerprint. Same input always gives the same hash.</p>
      <textarea v-model="inputText" placeholder="Type something to hash..." rows="3"></textarea>
      <button class="primary" @click="hashText">Generate Hash</button>
      <div v-if="hashResult" class="result">
        <label>SHA-256:</label>
        <code>{{ hashResult }}</code>
      </div>
    </div>

    <!-- ENCRYPT -->
    <div v-if="activeTab === 'encrypt'" class="panel">
      <h2>AES-GCM Encryption</h2>
      <p>Encrypts text with a randomly generated AES-256 key. You can decrypt it back in the same session.</p>
      <textarea v-model="encryptInput" placeholder="Type text to encrypt..." rows="3"></textarea>
      <div class="btn-row">
        <button class="primary" @click="encryptText">Encrypt</button>
        <button class="secondary" @click="decryptText" :disabled="!encryptedResult">Decrypt</button>
      </div>
      <div v-if="encryptedResult" class="result">
        <label>Encrypted (Base64):</label>
        <code class="break">{{ encryptedResult }}</code>
      </div>
      <div v-if="decryptedResult" class="result success">
        <label>Decrypted:</label>
        <code>{{ decryptedResult }}</code>
      </div>
    </div>

    <!-- RANDOM -->
    <div v-if="activeTab === 'random'" class="panel">
      <h2>Cryptographically Secure Random</h2>
      <p>Unlike <code>Math.random()</code>, this generates truly unpredictable values suitable for security purposes.</p>
      <button class="primary" @click="generateRandom">Generate 16 random bytes</button>
      <div v-if="randomResult" class="result">
        <label>Random (hex):</label>
        <code>{{ randomResult }}</code>
      </div>
    </div>

    <div v-if="error" class="error">⚠️ {{ error }}</div>
  </div>
</template>

<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body { background: #0d0d1a; color: #fff; font-family: sans-serif; }

.app {
  max-width: 700px;
  margin: 0 auto;
  padding: 40px 24px;
}

h1 { font-size: 2rem; color: #00b4d8; margin-bottom: 8px; }
.subtitle { color: #aaa; margin-bottom: 32px; }

.tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 24px;
}
.tabs button {
  padding: 10px 18px;
  border-radius: 8px;
  border: 1px solid #333;
  background: #1a1a2e;
  color: #aaa;
  cursor: pointer;
  font-size: 0.95rem;
  transition: all 0.2s;
}
.tabs button.active {
  background: #00b4d8;
  color: #fff;
  border-color: #00b4d8;
}

.panel {
  background: #1a1a2e;
  border-radius: 12px;
  padding: 28px;
}
.panel h2 { margin-bottom: 8px; font-size: 1.2rem; }
.panel p { color: #aaa; margin-bottom: 16px; font-size: 0.9rem; }

textarea {
  width: 100%;
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #333;
  background: #0d0d1a;
  color: #fff;
  font-size: 1rem;
  resize: vertical;
  margin-bottom: 12px;
}

.btn-row { display: flex; gap: 8px; margin-bottom: 4px; }

button.primary {
  padding: 10px 24px;
  background: #00b4d8;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  margin-bottom: 16px;
}
button.primary:hover { background: #0096c7; }
button.secondary {
  padding: 10px 24px;
  background: #2a2a3e;
  color: #fff;
  border: 1px solid #444;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  margin-bottom: 16px;
}
button.secondary:disabled { opacity: 0.4; cursor: default; }

.result {
  background: #0d0d1a;
  border-radius: 8px;
  padding: 14px;
  margin-top: 8px;
}
.result.success { border: 1px solid #2d6a4f; }
.result label { font-size: 0.8rem; color: #aaa; display: block; margin-bottom: 6px; }
.result code { font-size: 0.85rem; color: #00b4d8; word-break: break-all; }
.result code.break { word-break: break-all; }

.error {
  margin-top: 20px;
  padding: 12px 16px;
  background: #3a1a1a;
  border-radius: 8px;
  color: #ff6b6b;
}
</style>
