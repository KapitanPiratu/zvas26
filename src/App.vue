<script setup>
import AdminView from "./components/AdminView.vue";
import CheckpointView from "./components/CheckpointView.vue";

const searchParams = new URLSearchParams(window.location.search);
const isAdmin = searchParams.get("view") == "admin" ? true : false;
</script>

<template>
    <AdminView v-if="isAdmin" />
    <CheckpointView v-else />
</template>

<style scoped></style>
