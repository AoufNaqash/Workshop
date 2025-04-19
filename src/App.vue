<script setup>
import { ref, onMounted } from 'vue'
import PocketBase from 'pocketbase'

const hi = "My name is"
const isTrue = ref(true)
const API_LINK = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=KDH03"
const pb = new PocketBase('http://127.0.0.1:8090')
const name = ref("")
const number = ref(0)
const prayerTime = ref({})
const capitalCities = ref([
  "New York", "Los Angeles", "Chicago", "Houston", "Phoenix"
])
const todos=ref([])
const newTodo = ref("")
function increment() {
  number.value++
}

function startInterval() {
  setInterval(increment, 1000)
}

function submitText() {
  if (name.value.trim()) {
    capitalCities.value.push(name.value.trim())
    name.value = ""
  }
}

function getPrayerTime() {
  fetch(API_LINK)
    .then(result => result.json())
    .then(data => {
      prayerTime.value = data.prayerTime[0]
    })
}

async function getTodos() {
  todos.value = await pb.collection('todos').getFullList()
  console.log(todos)
}

onMounted(async () => {
  getTodos()
})

function addTodo() {
  const newTodo = {
    item: todoInput.value,
    isDone: false
  }

  pb.collection('todos').create(newTodo)
    .then((res) => {
      todos.value.push(res)
      todoInput.value = ""
    })
    .catch((err) => {
      console.error(err)
    })
}
</script>


<template>
  <div class="max-w-4xl mx-auto p-6">
    <h1 class="text-2xl font-bold mb-4 text-center">Todo List</h1>

    <div class="flex items-center gap-4 mb-6">
      <input
        type="text"
        v-model="newTodo"
        placeholder="Add a new todo..."
        class="flex-1 px-4 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        @click="addTodo"
        class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition"
      >
        Add
      </button>
      <button
        @click="getTodos"
        class="px-4 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600 transition"
      >
        Refresh
      </button>
    </div>

    <table class="w-full table-auto border-collapse">
      <thead>
        <tr class="bg-gray-100 text-left">
          <th class="border px-4 py-2">ID</th>
          <th class="border px-4 py-2">Todo</th>
          <th class="border px-4 py-2">Completed</th>
          <th class="border px-4 py-2">Created</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="todo in todos"
          :key="todo.id"
          class="hover:bg-gray-50 transition"
        >
          <td class="border px-4 py-2">{{ todo.id }}</td>
          <td class="border px-4 py-2">{{ todo.todo }}</td>
          <td class="border px-4 py-2">{{ todo.isDone ? 'DONE' : 'NOT DONE' }}</td>
          <td class="border px-4 py-2">{{ new Date(todo.created).toLocaleString() }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
