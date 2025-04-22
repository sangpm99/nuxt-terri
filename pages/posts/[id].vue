<script setup lang="ts">
definePageMeta({
  pageTransition: {
    name: "slide-right",
    mode: "out-in",
  },
  middleware(to, from) {
    if (to.meta.pageTransition && typeof to.meta.pageTransition !== "boolean")
      to.meta.pageTransition.name =
        +to.params.id! > +from.params.id! ? "slide-left" : "slide-right";
  },
});

const route = useRoute();
const id = computed(() => Number(route.params.id || 1));
const prev = computed(() => "/posts/" + (id.value - 1));
const next = computed(() => "/posts/" + (id.value + 1));
</script>

<template>
  <div>
    <h1>#{{ $route.params.id }}</h1>
    <div v-if="$route.params.id">
      <NuxtLink :to="prev">Prev</NuxtLink> |
      <NuxtLink :to="next">Next</NuxtLink>
    </div>
  </div>
</template>

<style>
.slide-left-enter-active,
.slide-left-leave-active,
.slide-right-enter-active,
.slide-right-leave-active {
  transition: all 0.2s;
}
.slide-left-enter-from {
  opacity: 0;
  transform: translate(50px, 0);
}
.slide-left-leave-to {
  opacity: 0;
  transform: translate(-50px, 0);
}
.slide-right-enter-from {
  opacity: 0;
  transform: translate(-50px, 0);
}
.slide-right-leave-to {
  opacity: 0;
  transform: translate(50px, 0);
}
</style>
