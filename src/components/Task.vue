<script setup>
import { ref } from "vue";

const { task } = defineProps(["task"]);
const taskId = task.id;

const completed = ref(false);

function toggleCompletion() {
    completed.value = !completed.value;
}

defineExpose({
    completed,
    taskId,
});
</script>

<template>
    <v-card
        @click="toggleCompletion"
        class="task-card"
        :class="{ 'task-completed': completed }"
    >
        <p>{{ task.description }}</p>
        <p>{{ task.points }} pts</p>
        <!-- <h3 v-if="completed">&check;</h3> -->
    </v-card>
</template>

<style scoped>
.task-card {
    min-height: 5rem;
    margin: 1rem;
    margin-left: 0;
    margin-right: 0;

    display: flex;
    justify-content: space-between;
    align-items: center;

    cursor: pointer;
}

.task-card p {
    margin: 1rem;
}

.task-completed {
    background-color: #06a77d;
    color: #03402f;
    text-decoration: line-through;
}

.task-completed p {
    opacity: 0.6;
}

h3 {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);

    font-size: 4rem;
    opacity: 0.9;
}
</style>
