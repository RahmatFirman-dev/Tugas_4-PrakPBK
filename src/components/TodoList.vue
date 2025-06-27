<template>
  <div>
    <h1> Daftar Pekerjaan yang di Lakukan </h1>
    <div class="todo-container">
      <div class="input-group">
        <input
          v-model="newTodo"
          @keyup.enter="addTodo"
          placeholder="Add new task..."
          class="todo-input"
        >
        <button @click="addTodo" class="add-button">Add</button>
      </div>

      <ul class="todo-list">
        <li 
          v-for="todo in filteredTodos" 
          :key="todo.id" 
          class="todo-item"
          :class="{
            completed: todo.completed,
            'priority-high': todo.priority === 'high',
            'priority-medium': todo.priority === 'medium',
            'priority-low': todo.priority === 'low'
          }"
        >
          <div class="todo-content">
            <input
              type="checkbox"
              v-model="todo.completed"
              @change="updateTodo(todo)"
              class="todo-checkbox"
              :id="'todo-'+todo.id"
            >
            <label :for="'todo-'+todo.id"></label>
            <span class="todo-text">{{ todo.title }}</span>
          </div>
          <div class="todo-actions">
            <select 
              v-model="todo.priority" 
              @change="updateTodo(todo)"
              class="priority-select"
            >
              <option value="high">Deadline sangat dekat</option>
              <option value="medium">Deadline santai</option>
              <option value="low">Deadline masih lama</option>
            </select>
            <button @click="deleteTodo(todo.id)" class="delete-button">
              Delete
            </button>
          </div>
        </li>
      </ul>

      <div class="stats">
        <span>{{ activeTodoCount }} items left</span>
        <button 
          v-if="completedTodoCount > 0" 
          @click="clearCompleted"
          class="clear-button"
        >
          Clear completed
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const API_URL = 'http://localhost:3000/todos'
const todos = ref([])
const newTodo = ref('')
const activeFilter = ref('all')


onMounted(async () => {
  try {
    const response = await axios.get(API_URL)
    todos.value = response.data
  } catch (error) {
    console.error('Error fetching todos:', error)
  }
})


const filteredTodos = computed(() => {
  switch (activeFilter.value) {
    case 'active': return todos.value.filter(todo => !todo.completed)
    case 'completed': return todos.value.filter(todo => todo.completed)
    default: return todos.value
  }
})

const activeTodoCount = computed(() => todos.value.filter(todo => !todo.completed).length)
const completedTodoCount = computed(() => todos.value.filter(todo => todo.completed).length)


const addTodo = async () => {
  if (!newTodo.value.trim()) return
  
  try {
    const response = await axios.post(API_URL, {
      title: newTodo.value.trim(),
      completed: false,
      priority: 'medium'
    })
    todos.value.push(response.data)
    newTodo.value = ''
  } catch (error) {
    console.error('Error adding todo:', error)
  }
}

const updateTodo = async (todo) => {
  try {
    await axios.put(`${API_URL}/${todo.id}`, todo)
  } catch (error) {
    console.error('Error updating todo:', error)
  }
}

const deleteTodo = async (id) => {
  try {
    await axios.delete(`${API_URL}/${id}`)
    todos.value = todos.value.filter(todo => todo.id !== id)
  } catch (error) {
    console.error('Error deleting todo:', error)
  }
}

const clearCompleted = () => {
  todos.value.forEach(todo => {
    if (todo.completed) deleteTodo(todo.id)
  })
}
</script>