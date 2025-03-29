<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

import ModalDialog from './ModalDialog.vue'
import { createConfirmDialog } from 'vuejs-confirm-dialog'
import NewTaskModal from './NewTaskModal.vue';

const tasks = ref<any[]>([])
const filteredTasks = ref<any[]>([])
const searchQuery = ref('')
const title = ref('')
const description = ref('')
const isModalOpen = ref(false)
const statusFilter = ref('all') // 'all', 'completed', 'pending'
const sortDirection = ref<'asc' | 'desc'>('desc')

const openNewTask = () => {
  isModalOpen.value = true;
}
const closeNewTask = () => {
  isModalOpen.value = false;
}
const handleNewTask = (data) => {
  title.value = data.title
  description.value = data.description
  closeNewTask();
  createTask();
}

const dialog = createConfirmDialog(ModalDialog)

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
  if(title.value == '' || description.value == '' || title.value == undefined || description.value == undefined) {
    return
  }
  await addTask({'title':title.value, 'description':description.value})
  toast.success("Added successfully!", {
        autoClose: 2000,
      });
  title.value = ''
  description.value = ''
  fetchTasks()
}

const removeTask = async (id: string) => {
  const { data, isCanceled } = await dialog.reveal()

  if(isCanceled) return

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
    <div class="d-flex align-items-center justify-content-between mb-4">
      <h1 class="display-4 mb-0 me-3">Task Management</h1>
      <button @click="openNewTask" class="btn btn-primary rounded-md px-4 d-flex justify-content-center align-items-center">
        New
      </button>
    </div>
    <NewTaskModal :isOpen="isModalOpen" @confirm="handleNewTask" @cancel="closeNewTask" />

    <!-- Search and Filter Controls -->
    <div class="row mb-3">
      <div class="col-md-9">
        <input v-model="searchQuery" type="text" class="form-control" placeholder="Search tasks..." @input="handleSearch" />
      </div>
      <div class="col-md-3">
        <select v-model="statusFilter" class="form-select" @change="handleStatusChange">
          <option value="all">All</option>
          <option value="completed">Completed</option>
          <option value="pending">Pending</option>
        </select>
      </div>
    </div>

    <!-- Task Grid (Table) -->
    <table class="table table-bordered table-hover">
      <thead class="table-dark">
        <tr>
          <th >Title</th>
          <th >Description</th>
          <th @click="toggleSort">Created At {{sortDirection == 'asc' ? '↑' : '↓'}}</th>
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