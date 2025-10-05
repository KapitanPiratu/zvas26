<script setup>
import { onMounted, ref } from "vue";

const apiUrl = import.meta.env.VITE_API_URL;
const showLoading = ref(false);

const snackbar = ref(false);
const snackbarMsg = ref("");

const teams = ref([]);

async function getTeams() {
    console.log("fetching");
    showLoading.value = true;
    await fetch(apiUrl + "/teams", {
        method: "GET",
    })
        .then((response) => response.json())
        .then((data) => {
            teams.value = data;
            showLoading.value = false;
        });
}

onMounted(() => {
    getTeams();

    setInterval(() => {
        getTeams();
    }, 10000);
});
</script>

<template>
    <nav>
        <h2>Admin panel</h2>
    </nav>

    <v-table>
        <thead>
            <tr>
                <th>Id</th>
                <th>Team</th>
                <th>Points</th>
                <th>Last Checkpoint</th>
                <th>Last Checkpoint Status</th>
            </tr>
        </thead>

        <tbody>
            <tr v-for="team in teams">
                <td>{{ team.id }}</td>
                <td>{{ team.name }}</td>
                <td>{{ team.points }}</td>
                <td>{{ "-" }}</td>
                <td>{{ "-" }}</td>
            </tr>
        </tbody>
    </v-table>

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

th {
    font-weight: bold !important;
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
