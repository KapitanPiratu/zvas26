<script setup>
import AdminView from "./components/AdminView.vue";
import AdminPath from "./components/AdminPath.vue";
import CheckpointView from "./components/CheckpointView.vue";
import CheckpointInit from "./components/CheckpointInit.vue";

const searchParams = new URLSearchParams(window.location.search);
const isAdmin = searchParams.get("view") == "admin" ? true : false;
const isAdmin2 = searchParams.get("view") == "admin2" ? true : false;
const checkpointKey = searchParams.get("key");
</script>

<template>
    <AdminView v-if="isAdmin" />
    <AdminPath v-else-if="isAdmin2" />
    <CheckpointInit v-else-if="checkpointKey" />
    <CheckpointView v-else />
</template>

<style scoped></style>
