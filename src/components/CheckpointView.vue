<script setup>
import { onBeforeUpdate, onMounted, ref } from "vue";
import Task from "./Task.vue";

const apiUrl = import.meta.env.VITE_API_URL;
const showLoading = ref(false);

const showError = ref(false);
const errorMsg = ref("");

const showSuccess = ref(false);

const searchParams = new URLSearchParams(window.location.search);
const teamId = searchParams.get("team") || "";
const id = searchParams.get("c");
if (id) localStorage.setItem("checkpoint", id);
const checkpointId = localStorage.getItem("checkpoint");
const checkpointKey = localStorage.getItem("key");
const checkpointName = localStorage.getItem("checkpoint_name");

const teams = ref([]);
const tasks = ref([]);
const teamName = ref("");

async function getTeams() {
    console.log("fetching");
    showLoading.value = true;
    await fetch(apiUrl + "/teams", {
        method: "GET",
    })
        .then((response) => response.json())
        .then((data) => {
            teams.value = data;
            // teams.value = data.map((team) => team.id);
            showLoading.value = false;

            if (teamId) {
                teamName.value = data.find((el) => el.id == teamId).name;
            }
        });
}

async function getTasks() {
    console.log("fetching");
    showLoading.value = true;
    await fetch(apiUrl + `/tasks/?c=${checkpointId}&key=${checkpointKey}`, {
        method: "GET",
    })
        .then((response) => response.json())
        .then((data) => {
            tasks.value = data;
            showLoading.value = false;
        });
}

onMounted(() => {
    if (typeof window !== "undefined") {
        const searchParams = new URLSearchParams(window.location.search);
        const teamId = searchParams.get("team") || "";
        const id = searchParams.get("c");

        // if (id) localStorage.setItem("checkpoint", id);
        // checkpointId = localStorage.getItem("checkpoint");
        // checkpointKey = localStorage.getItem("key");

        if (teamId) {
            teamModel.value = teamId;
            confirmCheckpoint();
        }
    }
    getTeams();
    getTasks();
});

// dummy key to force re-render for v-if to work correctly
const key = ref(0);

const showCard = ref(false);

const teamModel = ref();
function confirmCheckpoint() {
    if (teamModel.value) {
        teamName.value = teams.value.find((el) => el.id == teamModel.value);

        if (typeof teamName.value === "object") {
            teamName.value = teamName.value.name;
        }

        if (checkpointId) {
            showCard.value = true;
            postArrival();
        }
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

    await fetch(apiUrl + `/arrivallog/?key=${checkpointKey}`, {
        method: "POST",
        body: JSON.stringify({
            team: teamModel.value,
            checkpoint: checkpointId,
            status: "arrived",
        }),
        headers: {
            "Content-Type": "application/json",
        },
    }).then(async (response) => {
        showLoading.value = false;
        if (response.status != 200) {
            showError.value = true;
            errorMsg.value = await response.json();
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

    await fetch(apiUrl + `/taskslog/?key=${checkpointKey}`, {
        method: "POST",
        body: JSON.stringify({
            team: teamModel.value,
            checkpoint: checkpointId,
            tasks: tasksResult.value,
        }),
        headers: {
            "Content-Type": "application/json",
        },
    }).then(async (response) => {
        showLoading.value = false;
        if (response.status != 201) {
            showError.value = true;
            errorMsg.value = await response.json();
        } else {
            showCard.value = false;
            teamModel.value = undefined;

            showSuccess.value = true;
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
        <h2 v-if="checkpointId">Stanoviště {{ checkpointName }}</h2>
        <h2 v-else>ZVaS 2026</h2>
    </nav>

    <div v-if="checkpointId">
        <div v-if="!showCard">
            <v-select
                :key="key"
                v-model="teamModel"
                label="Choose team"
                :items="teams"
                item-title="name"
                item-value="id"
                class="select"
            ></v-select>
            <v-btn :key="key" @click="confirmCheckpoint" class="btn"
                >Confirm</v-btn
            >
        </div>

        <v-card v-else class="card">
            <h1>Tým {{ teamName }}</h1>
            <Task
                v-for="task in tasks"
                :key="task.id"
                :ref="setTaskRef"
                :task="task"
            />
            <v-btn @click="readChildValues">Odeslat</v-btn>
        </v-card>
    </div>

    <div v-else>
        <v-card class="card">
            <!-- <h1>Tým {{ teamName }}</h1> -->
            <h1>Nothing to see here...</h1>
        </v-card>
    </div>

    <div class="loading" v-if="showLoading">
        <v-progress-circular indeterminate></v-progress-circular>
    </div>

    <div class="success" v-if="showSuccess">
        <h1>Vše je v pořádku</h1>

        <p>Nyní můžeš zavřít toto okno.</p>
    </div>

    <div class="error" v-if="showError">
        <h1>Něco se pokazilo!</h1>

        <p>{{ errorMsg.error }}</p>
    </div>
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

.error {
    width: 100vw;
    height: 100vh;
    background-color: rgb(255, 0, 0, 0.85);

    position: fixed;
    top: 0;
    left: 0;

    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    color: rgb(58, 0, 0);
}

.error * {
    transform: translateY(-10vh);
}

.success {
    width: 100vw;
    height: 100vh;
    background-color: rgb(6, 167, 125, 0.85);

    position: fixed;
    top: 0;
    left: 0;

    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    color: #03402f;
}

.success * {
    transform: translateY(-10vh);
}
</style>
