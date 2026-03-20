<script setup>
import AdminTable from "./AdminTable.vue";
import AdminDialog from "./AdminDialog.vue";
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

// logic for sending teams
const showDialog = ref(false);

function toggleDialog() {
    showDialog.value = !showDialog.value;
}

async function postTeam(data) {
    showLoading.value = true;
    const { name, organization, path } = data;

    console.log("blabla", name, organization, path);

    await fetch(apiUrl + `/newteam`, {
        method: "POST",
        body: JSON.stringify(data),
        headers: {
            "Content-Type": "application/json",
        },
    }).then((response) => {
        showLoading.value = false;
        if (response.status != 201) {
            snackbarMsg.value = `Something went really wrong - TEAM WAS NOT CREATED - status code: ${response.status}`;
            snackbar.value = true;
        } else {
            showDialog.value = false;
        }
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
        <v-btn class="nav-btn" @click="toggleDialog">Send team</v-btn>
    </nav>

    <AdminTable :teams="teams" />
    <AdminDialog
        @close-dialog="showDialog = false"
        @post-team="postTeam"
        v-if="showDialog"
    />

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

.nav-btn {
    margin-right: 2rem;
    position: absolute;
    right: 0;
    top: auto;
}
</style>
