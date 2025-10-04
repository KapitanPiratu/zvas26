<script setup>
let test = true;
</script>

<template>
    <button @click="test = !test">btn</button>
    <h3 v-if="test">Admin</h3>
</template>

<style scoped></style>
