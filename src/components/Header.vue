<script setup>

defineProps({
  msg: String
})

</script>

<template>
  <h1>{{ msg }}</h1>

  <p>
   Play chess online!
  </p>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
