<script setup lang="ts">
async function addTodo() {
  const todo = await $fetch("/api/todos", {
    method: "GET",
  });
  console.log(todo);
}

onMounted(async () => {
  await addTodo();
});
</script>

<template>
  <NuxtLink to="/about">Go to Blue</NuxtLink>
  <NuxtLoadingIndicator></NuxtLoadingIndicator>
</template>
