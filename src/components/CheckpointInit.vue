<script setup>
import { ref, onMounted } from "vue";

const searchParams = new URLSearchParams(window.location.search);
const apiUrl = import.meta.env.VITE_API_URL;
const showLoading = ref(true);
const msg = ref("");
const status = ref(false);

async function postCheckpointKey(key) {
    showLoading.value = true;

    await fetch(apiUrl + "/checkpoint", {
        method: "POST",
        body: JSON.stringify({
            key: key,
        }),
        headers: {
            "Content-Type": "application/json",
        },
    })
        .then((response) => {
            showLoading.value = false;
            if (response.status != 200) {
                msg.value = "Something went wrong";
            } else {
                status.value = true;
            }
            return response;
        })
        .then((response) => response.json())
        .then((data) => {
            if (status.value) {
                msg.value = data;
                localStorage.setItem("key", key);
                localStorage.setItem("checkpoint", data.id);
                localStorage.setItem("checkpoint_name", data.name);
            }
        });
}

onMounted(() => {
    postCheckpointKey(searchParams.get("key"));
});
</script>

<template>
    <nav>
        <h2>Checkpoint {{ checkpointId }}</h2>
    </nav>

    <div class="wrapper">
        <h1 v-if="!status && msg">{{ msg }}</h1>
        <v-card v-if="status" class="card">
            <h1>Vše je v&nbsp;pořádku</h1>
            <p>
                Tvé stanoviště:
                <span class="checkpoint-name">{{ msg.name }}</span>
            </p>
            <h2>Nyní můžeš zavřít toto okno :D</h2>
        </v-card>
    </div>

    <div class="loading" v-if="showLoading">
        <v-progress-circular indeterminate></v-progress-circular>
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

.wrapper {
    width: 100vw;
    height: 100vh;

    display: flex;

    justify-content: center;
    align-items: center;
}

.card {
    width: 80vw;
    height: 80vh;

    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: center;

    text-align: center;
}

.card h1 {
    color: #06a77d;
    font-size: 4rem;
    margin-top: 15vh;
}

.card p {
    font-size: 1.1rem;
}

.checkpoint-name {
    font-weight: bold;
}

.card h2 {
    margin: 1rem;
    margin-top: 6rem;
    font-weight: 590;
    font-size: 1.1rem;
    /* text-decoration: underline; */
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
</style>
