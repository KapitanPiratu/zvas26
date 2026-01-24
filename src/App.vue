<script setup>
import AdminView from "./components/AdminView.vue";
import CheckpointView from "./components/CheckpointView.vue";
import CheckpointInit from "./components/CheckpointInit.vue";

const searchParams = new URLSearchParams(window.location.search);
const isAdmin = searchParams.get("view") == "admin" ? true : false;
const checkpointKey = searchParams.get("key");
</script>

<template>
    <AdminView v-if="isAdmin" />
    <CheckpointInit v-else-if="checkpointKey" />
    <CheckpointView v-else />
</template>

<style scoped></style>
