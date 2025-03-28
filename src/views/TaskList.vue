<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const tasks = ref<any[]>([])
const filteredTasks = ref<any[]>([])
const newTask = ref({ title: '', description: '' })
const searchQuery = ref('')
const statusFilter = ref('all') // 'all', 'completed', 'pending'
const sortDirection = ref<'asc' | 'desc'>('asc')

const API_URL = 'http://127.0.0.1:5136/api/task' // Update with your backend URL

const addTask = async (task: { title: string; description: string }) => {
  const response = await axios.post(API_URL, task)
  return response.data
}

const deleteTask = async (id: string) => {
  await axios.delete(`${API_URL}/${id}`)
}

const fetchTasks = async () => {
  axios.get(API_URL).then(res => {
    tasks.value = res.data
    console.log("Fetching", res.data)
    applyFilters()
  }).catch(() => {
    tasks.value = filteredTasks.value = []
  })
}

const createTask = async () => {
  if(newTask.title == '' || newTask.description == '')
    return
  await addTask(newTask.value)
  newTask.value = { title: '', description: '' }
  fetchTasks()
}

const removeTask = async (id: string) => {
  await deleteTask(id)
  fetchTasks()
}

// Apply filters and sorting
const applyFilters = () => {
  filteredTasks.value = tasks.value
    .filter(task => {
      const matchesSearch = task.title.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
                           task.description.toLowerCase().includes(searchQuery.value.toLowerCase())

      const matchesStatus =
        statusFilter.value === 'all' ||
        (statusFilter.value === 'completed' && task.isCompleted) ||
        (statusFilter.value === 'pending' && !task.isCompleted)

      return matchesSearch && matchesStatus
    })
    .sort((a, b) => {
      const dateA = new Date(a.createdAt).getTime()
      const dateB = new Date(b.createdAt).getTime()
      return sortDirection.value === 'asc' ? dateA - dateB : dateB - dateA
    })
}

// Watch for changes and apply filters
const handleSearch = () => applyFilters()
const handleStatusChange = () => applyFilters()
const toggleSort = () => {
  sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
  applyFilters()
}

onMounted(fetchTasks)
</script>

<template>
  <div class="container py-5">
    <h1 class="display-4 mb-4">Task Management</h1>

    <!-- Search and Filter Controls -->
    <div class="row mb-3">
      <div class="col-md-6">
        <input v-model="searchQuery" type="text" class="form-control" placeholder="Search tasks..." @input="handleSearch" />
      </div>
      <div class="col-md-3">
        <select v-model="statusFilter" class="form-select" @change="handleStatusChange">
          <option value="all">All</option>
          <option value="completed">Completed</option>
          <option value="pending">Pending</option>
        </select>
      </div>
      <div class="col-md-3">
        <button class="btn btn-secondary w-100" @click="toggleSort">
          Sort by Date ({{ sortDirection === 'asc' ? 'Oldest' : 'Newest' }})
        </button>
      </div>
    </div>

    <!-- Add Task -->
    <div class="row mb-3">
      <div class="col-md-4">
        <input v-model="newTask.title" placeholder="Task title" class="form-control" />
      </div>
      <div class="col-md-4">
        <input v-model="newTask.description" placeholder="Task description" class="form-control" />
      </div>
      <div class="col-md-4">
        <button @click="createTask" class="btn btn-primary w-100">Add Task</button>
      </div>
    </div>

    <!-- Task Grid (Table) -->
    <table class="table table-bordered table-hover">
      <thead class="table-dark">
        <tr>
          <th>Title</th>
          <th>Description</th>
          <th>Created At</th>
          <th>Status</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="task in filteredTasks" :key="task.id">
          <td>{{ task.title }}</td>
          <td>{{ task.description }}</td>
          <td>{{ new Date(task.createdAt).toLocaleString() }}</td>
          <td>
            <span :class="task.isCompleted ? 'badge bg-success' : 'badge bg-warning'">
              {{ task.isCompleted ? 'Completed' : 'Pending' }}
            </span>
          </td>
          <td>
            <button @click="removeTask(task.id)" class="btn btn-danger btn-sm">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- No Tasks Message -->
    <div v-if="filteredTasks.length === 0" class="text-center mt-3">
      <p class="text-muted">No tasks found.</p>
    </div>
  </div>
</template>