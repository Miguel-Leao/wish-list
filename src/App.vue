<script setup>
import { ref, computed, onMounted, watch } from 'vue'

let id = 0

const newWish = ref('')
const hideCompleted = ref(false)
const wishes = ref([])
const loading = ref(true)

const trimmedText = computed(() => newWish.value.trim())
const normalizedText = computed(() => trimmedText.value.toLowerCase())

const isDuplicate = computed(() =>
  wishes.value.some(w => w.text.toLowerCase() === normalizedText.value)
)

const errorMessage = computed(() => {
  if (trimmedText.value.length === 0) {
    return 'O campo não pode estar vazio!'
  }
  if (trimmedText.value.length < 3) {
    return 'O desejo deve ter pelo menos 3 letras!'
  }
  if (isDuplicate.value) {
    return 'Este desejo já existe na lista!'
  }
  return ''
})

const filteredWishes = computed(() => {
  return hideCompleted.value
    ? wishes.value.filter((t) => !t.done)
    : wishes.value
})

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

async function loadWishes() {
  await delay(2000)
  const saved = localStorage.getItem('wishes')
  if (saved) {
    wishes.value = JSON.parse(saved)
    id = wishes.value.length > 0 ? Math.max(...wishes.value.map(w => w.id)) + 1 : 0
  }
  loading.value = false
}

function addWish() {
  if (errorMessage.value) return

  wishes.value.push({ id: id++, text: trimmedText.value, done: false })
  newWish.value = ''
}

function removeWish(wish) {
  wishes.value = wishes.value.filter((t) => t !== wish)
}

watch(wishes, (newWishes) => {
  localStorage.setItem('wishes', JSON.stringify(newWishes))
}, { deep: true })

onMounted(() => {
  loadWishes()
})
</script>


<template>
  <div class="container">
    <div class="card">
      <div v-if="loading" class="loading-text">Carregando seus desejos...</div>
      <div v-else>
        <form @submit.prevent="addWish">
          <input
            v-model.trim="newWish"
            required
            placeholder="New wish"
          />
          <button :disabled="!!errorMessage">Add Wish</button>
        </form>

        <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>

        <transition-group name="fade" tag="ul">
          <li v-for="wish in filteredWishes" :key="wish.id">
            <input type="checkbox" v-model="wish.done" />
            <span :class="{ done: wish.done }">{{ wish.text }}</span>
            <button @click="removeWish(wish)">X</button>
          </li>
        </transition-group>

        <button @click="hideCompleted = !hideCompleted">
          {{ hideCompleted ? 'Show All' : 'Hide completed' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style>

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  background-color: #f0f2f5;
  padding: 20px;
}

.container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  background-color: #ffffff;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 500px;
}

form {
  margin-bottom: 10px;
  display: flex;
  gap: 10px;
}

input[type="text"], input[placeholder="New wish"] {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 12px;
  border: none;
  background-color: #4caf50;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:disabled {
  background-color: #9e9e9e;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  background-color: #45a049;
}

ul {
  list-style: none;
  padding: 0;
  margin-top: 10px;
}

li {
  background-color: #f9f9f9;
  padding: 10px 12px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

li input[type="checkbox"] {
  margin-right: 10px;
}

.done {
  text-decoration: line-through;
  color: #888;
}

li button {
  background-color: #f44336;
  padding: 5px 10px;
  border-radius: 4px;
  color: white;
}

li button:hover {
  background-color: #d32f2f;
}

.error-message {
  color: #f44336;
  font-size: 0.9rem;
  margin-bottom: 10px;
  padding-left: 4px;
}

.loading-text {
  text-align: center;
  font-size: 1.1rem;
  color: #555;
  padding: 20px 0;
}

.fade-enter-active, .fade-leave-active {
  transition: all 0.6s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: scale(0.8);
  transform: translateX(20px);
  background-color: #328d37;
}

.fade-enter-to, .fade-leave-from {
  opacity: 1;
  transform: scale(1);
  transform: translateX(0);
  background-color: #ffffff;
}

</style>