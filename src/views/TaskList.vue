<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const tasks = ref<any[]>([])
const filteredTasks = ref<any[]>([])
const newTask = ref({ title: '', description: '' })

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
    console.log(res.data)
    tasks.value = res.data
    filteredTasks.value = res.data
  }).catch(err => {
    tasks.value = filteredTasks.value = []
  })
}

const createTask = async () => {
  await addTask(newTask.value)
  newTask.value = { title: '', description: '' }
  fetchTasks()
}

const removeTask = async (id: string) => {
  await deleteTask(id)
  fetchTasks()
}
const handleSearch = async (e) => {
  const searchValue = e.target.value.toLowerCase()
  filteredTasks.value = await tasks.value.filter(task => {
    return task.title.toLowerCase().indexOf(searchValue) >= 0 || task.description.toLowerCase().indexOf(searchValue) >= 0
  })
}

onMounted(fetchTasks)
</script>

<template>
  <div class="container py-5">
    <h1 class="display-4 mb-4">Task Management</h1>

    <div class="mb-4">
      <input type="text" class="form-control" placeholder="Search tasks..." @input="handleSearch" />
    </div>

    <div class="mb-4 d-flex flex-row gap-2">
      <input v-model="newTask.title" placeholder="Task title" class="form-control mb-2" />
      <input v-model="newTask.description" placeholder="Task description" class="form-control mb-2" />
      <button @click="createTask" class="btn btn-primary">Add Task</button>
    </div>

    <ul class="list-unstyled">
      <li v-for="task in filteredTasks" :key="task.id" class="border p-3 mb-2 d-flex gap-2 justify-content-between">
        <div class="d-flex flex-column">
          <span>{{ task.title }} - {{ task.description }}</span>
          <span>{{ "Created at: "+task.createdAt }}</span>
        </div>
        <div class="flex-row flex-row ">
          <button @click="removeTask(task.id)" class="btn btn-danger">Delete</button>
        </div>
      </li>
    </ul>
  </div>

</template>
