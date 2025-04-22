<script setup>
import { useRoute } from "vue-router";

const route = useRoute();
</script>

<template>
  <div class="wrapper">
    <!-- Hộp nằm ngoài content -->
    <div
      :class="['box', route.path === '/' ? 'red' : 'pink']"
      view-transition-name="color-box"
    ></div>

    <!-- Phần nội dung sẽ thay đổi -->
    <NuxtPage />
  </div>
</template>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 50px;
}
.box {
  width: 200px;
  height: 200px;
  margin-bottom: 20px;
  border-radius: 16px;
  transition: background-color 0.5s ease;
}
.red {
  background-color: red;
}
.pink {
  background-color: pink;
}
</style>
