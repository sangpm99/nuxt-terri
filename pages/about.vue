<script setup lang="ts">
const color = ref("red")
</script>

<template>
  <div class="text">hello</div>
</template>

<style>
.text {
  color: v-bind(color);
}
</style>
