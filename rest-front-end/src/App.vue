<template>
  <div>
    <RouterLink to="/">Home</RouterLink> |
    <RouterLink to="/products">Products</RouterLink>
  </div>
    <RouterView />
</template>

<script>
export default {
  name: 'App'
};
</script>

<style scoped>
</style>