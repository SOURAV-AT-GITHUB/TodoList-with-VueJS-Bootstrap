<script setup>
import { ref, onMounted, watch } from 'vue'
const tasks = ref([])
const filterTasks = ref([])
let isLoading = ref(true)
let filter = ref('All')
let sort = ref(false)
const handleSubmit = (event) => {
  event.preventDefault()
  let obj = {
    id: Date.now(),
    task: event.target[0].value,
    completed: false
  }
  tasks.value.push(obj)
  localStorage.setItem('tasks', JSON.stringify(tasks.value))
  event.target[0].value = ''
}
const handleStatusChange = (task) => {
  task.completed = !task.completed
  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}
const handleDelete = (id) => {
  tasks.value = tasks.value.filter((task) => task.id != id)
  filterTasks.value = filterTasks.value.filter((task) => task.id != id)
  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}
const getCompletedTaskCount = () => {
  let count = 0
  tasks.value.forEach((task) => {
    if (task.completed) count++
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
const getData = async () => {
  if (localStorage.getItem('tasks')) {
    filterTasks.value = tasks.value = JSON.parse(localStorage.getItem('tasks'))
    isLoading.value = false
  } else {
    try {
      const response = await fetch('https://run.mocky.io/v3/16c30903-3570-44ea-a0e8-848cbeffc3a6')
      let text = await response.text()
      filterTasks.value = tasks.value = JSON.parse(fixData(text))
      localStorage.setItem('tasks', JSON.stringify(tasks.value))
    } catch (error) {
      console.log(error)
      alert('Error getting data from server')
      isLoading.value = false
    } finally {
      isLoading.value = false
    }
  }
}
const handleSort = ()=>{
  console.log(sort.value);
  
  if (sort.value===true){
    filterTasks.value.sort((a,b)=>{
      if(a.taks > b.taks) return 1
      if(a.task <b.task) return -1
      // sort.value = false
      return 0
    }) 
  }
   if(sort.value ===false){
    filterTasks.value.sort((a,b)=>{
      if(a.taks < b.taks) return 1
      if(a.task >b.task) return -1
      // sort.value = true
      return 0
    })
  }
  sort.value =!sort.value
}
watch(filter, (newValue) => {
  if (filter.value === 'All') {
    filterTasks.value = tasks.value
  } else if (filter.value === 'Pending') {
    filterTasks.value = tasks.value.filter((e) => e.completed === false)
  } else if (filter.value === 'Completed') {
    filterTasks.value = tasks.value.filter((e) => e.completed === true)
  }
})
watch(tasks,(newValue)=>{
  if (filter.value === 'All') {
    filterTasks.value = tasks.value
  } else if (filter.value === 'Pending') {
    filterTasks.value = tasks.value.filter((e) => e.completed === false)
  } else if (filter.value === 'Completed') {
    filterTasks.value = tasks.value.filter((e) => e.completed === true)
  }
})
onMounted(getData)
</script>

<template>
  <form @submit="handleSubmit">
    <input
      type="text"
      placeholder="Enter Task"
      class="rounded text-center w-auto fw-bold fs-4"
      required
      minlength="3"
    />
    <br />
    <button class="btn btn-success my-3 px-4">Add Task</button>
  </form>
  <button @click="(e)=>handleSort()" type='button'>Sort</button>
  <label class="fs-3">Filter :&nbsp; </label>
  <select class="fs-4 rounded text-center" @change="(e) => (filter = e.target.value)">
    <option value="All">All</option>
    <option value="Pending">Pending</option>
    <option value="Completed">Completed</option>
  </select>
  <h1 v-if="isLoading">Loading</h1>

  <div v-else-if="filterTasks.length > 0">
    <h3 class="mb-2">Total Tasks : {{ tasks.length }}</h3>
    <h3>Completed Tasks : {{ getCompletedTaskCount() }}</h3>
    <table class="table table-bordered mt-3">
      <thead>
        <tr>
          a
          <th scope="col">#</th>
          <th scope="col">Task</th>
          <th scope="col">Status</th>
          <th scope="col">Action</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(task, index) in filterTasks"
          :key="task.id"
          v-show="
            filter == 'All' ||
            (filter == 'Pending' && !task.completed) ||
            (filter == 'Completed' && task.completed)
          "
        >
          <td>{{ index + 1 }}</td>
          <td
            :style="{
              textDecoration: task.completed ? 'line-through' : 'none',
              textDecorationColor: 'red'
            }"
            class="fs-4"
          >
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
