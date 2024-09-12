<script setup>
const handleSubmit = (event) => {
  event.preventDefault()
  console.log(event)
}
</script>

<template>
 
<form @submit="handleSubmit">
  <input type="text" placeholder="Enter Task">
  <button class="btn btn-success">Add Task</button>

</form>
</template>

<style scoped>

</style>
