<script setup>
import { onBeforeUpdate, onMounted, ref } from "vue";
import Task from "./Task.vue";

const apiUrl = import.meta.env.VITE_API_URL;
const showLoading = ref(false);

const snackbar = ref(false);
const snackbarMsg = ref("");

//display checkpoint id (from url params)
const searchParams = new URLSearchParams(window.location.search);
const checkpointId = searchParams.get("c") || "";
const teamId = searchParams.get("team") || "";

const teams = ref([]);
const tasks = ref([]);

async function getTeams() {
    console.log("fetching");
    showLoading.value = true;
    await fetch(apiUrl + "/teams", {
        method: "GET",
    })
        .then((response) => response.json())
        .then((data) => {
            teams.value = data.map((team) => team.id);
            showLoading.value = false;
        });
}

async function getTasks() {
    console.log("fetching");
    showLoading.value = true;
    await fetch(apiUrl + `/tasks/?c=${checkpointId}`, {
        method: "GET",
    })
        .then((response) => response.json())
        .then((data) => {
            tasks.value = data;
            showLoading.value = false;
        });
}

onMounted(() => {
    getTeams();
    getTasks();

    if (teamId) {
        teamModel.value = teamId;
        confirmCheckpoint()
    }
});

// dummy key to force re-render for v-if to work correctly
const key = ref(0);

const showCard = ref(false);

const teamModel = ref();
function confirmCheckpoint() {
    if (teamModel.value) {
        showCard.value = true;
        postArrival();
    }
    key.value += 1;
}

//reading data from Task component
const taskRefs = ref([]);
const tasksResult = ref([]);

const setTaskRef = (el) => {
    if (el) taskRefs.value.push(el);
};

onBeforeUpdate(() => {
    taskRefs.value = [];
});

async function postArrival() {
    showLoading.value = true;

    await fetch(apiUrl + "/arrivallog", {
        method: "POST",
        body: JSON.stringify({
            team: teamModel.value,
            checkpoint: checkpointId,
            status: "arrived",
        }),
        headers: {
            "Content-Type": "application/json",
        },
    }).then((response) => {
        showLoading.value = false;
        if (response.status != 200) {
            snackbarMsg.value = `Something went really wrong - ARRIVAL WAS NOT LOGGED - status code: ${response.status}`;
            snackbar.value = true;
        }
    });
}

async function postTasks() {
    /**
     * Posts tasks to server.
     *
     * Sends team id, checkpoint id and tasks for them to be logged to the db.
     * If server returns status code other than 201, snackbar with error message is shown.
     */

    showLoading.value = true;

    await fetch(apiUrl + "/taskslog", {
        method: "POST",
        body: JSON.stringify({
            team: teamModel.value,
            checkpoint: checkpointId,
            tasks: tasksResult.value,
        }),
        headers: {
            "Content-Type": "application/json",
        },
    }).then((response) => {
        showLoading.value = false;
        if (response.status != 201) {
            snackbarMsg.value = `Something went really wrong - DATA WAS NOT LOGGED - status code: ${response.status}`;
            snackbar.value = true;
        } else {
            showCard.value = false;
            teamModel.value = undefined;

            snackbarMsg.value = "Data succesfully submited";
            snackbar.value = true;
            setTimeout(() => (snackbar.value = false), 5000);
        }
    });
}

const readChildValues = () => {
    /**
     * Reads values (id and completed status) from Task components.
     */

    tasksResult.value = [];
    tasks.value.map((t) => {
        tasksResult.value.push({
            id: t.id,
            completed: taskRefs.value.find((el) => el.taskId == t.id).completed,
        });
    });

    postTasks();
};
</script>

<template>
    <nav>
        <h2>Checkpoint {{ checkpointId }}</h2>
    </nav>

    <div v-if="!showCard">
        <v-select
            :key="key"
            v-model="teamModel"
            label="Choose team"
            :items="teams"
            class="select"
        ></v-select>
        <v-btn :key="key" @click="confirmCheckpoint" class="btn">Confirm</v-btn>
    </div>

    <v-card v-else class="card">
        <h1>{{ teamModel }}</h1>
        <Task
            v-for="task in tasks"
            :key="task.id"
            :ref="setTaskRef"
            :task="task"
        />
        <v-btn @click="readChildValues">CLICK</v-btn>
    </v-card>

    <div class="loading" v-if="showLoading">
        <v-progress-circular indeterminate></v-progress-circular>
    </div>

    <v-snackbar class="snackbar" v-model="snackbar">
        {{ snackbarMsg }}
        <template v-slot:actions>
            <v-btn @click="snackbar = false">close</v-btn>
        </template>
    </v-snackbar>
</template>

<style scoped>
nav {
    width: 100%;
    height: 5rem;
    background-color: #3172b6;

    display: flex;
    align-items: center;
}

nav h2 {
    color: #fff;
    margin-left: 2rem;
}

.select,
.btn {
    margin-left: 2rem;
    margin-right: 2rem;
    max-width: 20rem;
}

.select {
    margin-top: 1rem;
}

.card {
    margin: 1rem;
    padding: 1rem;
}

.loading {
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;

    display: flex;
    justify-content: center;
    align-items: center;

    background-color: #fff;
    opacity: 0.8;
}

.snackbar {
    position: absolute;
    top: 5%;
    left: 50%;
    transform: translate(-50%, -50%);
}
</style>
