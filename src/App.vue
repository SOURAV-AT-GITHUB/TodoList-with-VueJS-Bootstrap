<script setup>
import { ref, onMounted } from 'vue'
const tasks = ref([])
let isLoading = ref(true)
let filter = ref('All')
const handleSubmit = (event) => {
  event.preventDefault()
  let obj = {
    id: Date.now(),
    task: event.target[0].value,
    completed: false
  }
  tasks.value.push(obj)
  event.target[0].value = ''
}
const handleStatusChange = (task) => {
  task.completed = !task.completed
}
const handleDelete = (id) => {
  tasks.value = tasks.value.filter((task) => task.id != id)
}
const getCompletedTaskCount=()=>{
  let count = 0
  tasks.value.forEach(task=>{
    if(task.completed) count++
  })
  return count
}
const fixData = (text) => {
  // Replace single quotes with double quotes for string values
  text = text.replace(/'/g, '"')
  // Add double quotes around property names
  text = text.replace(/([{,])\s*(\w+):/g, '$1 "$2":')
  // Fix any trailing commas
  text = text.replace(/,(\s*[}\]])/g, '$1')
  return text
}
const fetchData = async () => {
  try {
    const response = await fetch('https://run.mocky.io/v3/16c30903-3570-44ea-a0e8-848cbeffc3a6')
    let text = await response.text()
    tasks.value = JSON.parse(fixData(text))
  } catch (error) {
    console.log(error)
    alert('Error getting data from server')
    isLoading.value = false
  } finally {
    isLoading.value = false
  }
}
onMounted(fetchData)
</script>

<template>
  <form @submit="handleSubmit">
    <input
      type="text"
      placeholder="Enter Task"
      class="rounded text-center w-auto fw-bold fs-4"
      required
    />
    <br />
    <button class="btn btn-success my-3 px-4">Add Task</button>
  </form>
  <h1 v-if="isLoading">Loading</h1>

  <div v-else-if="tasks.length > 0">
    <h3 class="mb-2">Total Tasks : {{ tasks.length }}</h3>
    <h3>Completed Tasks : {{ getCompletedTaskCount() }}</h3>
    <label class="fs-3">Filter :&nbsp; </label>
    <select class="fs-4 rounded text-center" @change="(e) => (filter = e.target.value)">
      <option value="All">All</option>
      <option value="Pending">Pending</option>
      <option value="Completed">Completed</option>
    </select>
    <table class="table table-bordered mt-3">
      <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Task</th>
          <th scope="col">Status</th>
          <th scope="col">Action</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(task, index) in tasks"
          :key="task.id"
          v-show="
            filter == 'All' ||
            (filter == 'Pending' && !task.completed) ||
            (filter == 'Completed' && task.completed)
          "
        >
          <td>{{ index + 1 }}</td>
          <td :style="{ textDecoration: task.completed ? 'line-through' : 'none',textDecorationColor:'red' }" class="fs-4">
            {{ task.task }}
          </td>
          <td>
            <input
              type="checkbox"
              :checked="task.completed"
              @change="() => handleStatusChange(task)"
              class="checkbox"
            />
          </td>
          <td>
            <button type="button" @click="() => handleDelete(task.id)" class="btn btn-danger">
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <h3 v-else class="mt-2">No tasks available</h3>
</template>


