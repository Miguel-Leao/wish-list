<script setup>
import { ref, computed, watch, onMounted } from 'vue'

let id = 0

const newWish = ref('')
const hideCompleted = ref(false)
const wishes = ref([])

onMounted(() => {
  const saved = localStorage.getItem('wishes')
  if (saved) {
    wishes.value = JSON.parse(saved)
    id = wishes.value.length > 0 ? Math.max(...wishes.value.map(w => w.id)) + 1 : 0
  }
})

watch(wishes, (newWishes) => {
  localStorage.setItem('wishes', JSON.stringify(newWishes))
}, { deep: true })

const filteredWishes = computed(() => {
  return hideCompleted.value
    ? wishes.value.filter((t) => !t.done)
    : wishes.value
})

function addWish() {
  wishes.value.push({ id: id++, text: newWish.value, done: false })
  newWish.value = ''
}

function removeWish(wish) {
  wishes.value = wishes.value.filter((t) => t !== wish)
}
</script>

<template>
  <form @submit.prevent="addWish">
    <input v-model="newWish" required placeholder="New wish" />
    <button>Add Wish</button>
  </form>

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
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  background-color: #f9f9f9;
  padding: 20px;
}

form {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
}

input[type="text"] {
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

button:hover {
  background-color: #45a049;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  background-color: #fff;
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
}

li button:hover {
  background-color: #d32f2f;
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
