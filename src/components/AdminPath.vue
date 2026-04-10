<script setup>
import { computed, ref, onMounted } from "vue";

const apiUrl = import.meta.env.VITE_API_URL;
const showLoading = ref(false);

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

// Total number of checkpoints as defined in your requirements
const TOTAL_CHECKPOINTS = 10;

// 1. Filter out teams that have finished (path is empty)
const finishedTeams = computed(() => {
    return teams.value.filter((team) => !team.path || team.path.length === 0);
});

// 2. Map the 10 checkpoints and find which active teams are at each one
const timeline = computed(() => {
    const checkpoints = [];

    for (let i = 1; i <= TOTAL_CHECKPOINTS; i++) {
        // Find teams whose latest log (index 0) matches this checkpoint ID
        // AND they are not "finished"
        const teamsAtThisPoint = teams.value.filter((team) => {
            const isNotFinished = team.path && team.path.length > 0;
            const latestLog = team.logs[0];
            return isNotFinished && latestLog && latestLog.checkpoint_id === i;
        });

        checkpoints.push({
            id: i,
            teams: teamsAtThisPoint,
        });
    }
    return checkpoints;
});

onMounted(() => {
    getTeams();

    setInterval(() => {
        getTeams();
    }, 10000);
});
</script>

<template>
    <nav>
        <h2>Track Progress</h2>
    </nav>

    <div class="race-container">
        <v-card class="timeline-card">
            <div class="timeline">
                <div
                    v-for="cp in timeline"
                    :key="cp.id"
                    class="checkpoint-node"
                >
                    <div class="checkpoint-label">Checkpoint {{ cp.id }}</div>

                    <div class="teams-here">
                        <div
                            v-for="team in cp.teams"
                            :key="team.id"
                            class="team-badge"
                            :class="team.logs[0].status"
                        >
                            <strong>{{ team.name }} </strong>
                            <span class="status-text">
                                ({{ team.logs[0].status }})
                                {{
                                    team.path_all == "1234567890" ? "⬇️" : "⬆️"
                                }}
                            </span>
                        </div>
                        <div v-if="cp.teams.length === 0" class="empty-node">
                            Clear
                        </div>
                    </div>
                </div>
            </div>
        </v-card>

        <v-card class="finished-card" v-if="finishedTeams.length > 0">
            <h3>Completed</h3>
            <ul>
                <li v-for="team in finishedTeams" :key="team.id">
                    🏆 {{ team.name }}
                </li>
            </ul>
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
    margin-bottom: 1rem;
}

nav h2 {
    color: #fff;
    margin-left: 2rem;
}

.race-container {
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.timeline-card,
.finished-card {
    padding: 1.5rem;
}

.checkpoint-node {
    border-left: 3px solid #eee;
    margin-left: 20px;
    padding: 15px 25px;
    position: relative;
}

.checkpoint-node::before {
    content: "";
    width: 16px;
    height: 16px;
    background: #3172b6;
    border-radius: 50%;
    position: absolute;
    left: -10px;
    top: 18px;
    border: 3px solid #fff;
    box-shadow: 0 0 0 2px #eee;
}

.checkpoint-label {
    font-weight: bold;
    color: #666;
    margin-bottom: 8px;
    font-size: 1.1em;
}

.teams-here {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

.team-badge {
    padding: 6px 12px;
    border-radius: 50px;
    font-size: 0.9em;
    display: flex;
    align-items: center;
    gap: 6px;
}

.status-text {
    font-size: 0.85em;
    opacity: 0.8;
}

.arrived {
    background-color: #367ac4;
    color: #122c49;
}

.departed {
    background-color: #06a77d;
    color: #03402f;
}

.empty-node {
    color: #bbb;
    font-style: italic;
    font-size: 0.9em;
}

.finished-card h3 {
    margin-bottom: 1rem;
    color: #3172b6;
}

.finished-card ul {
    list-style: none;
    padding: 0;
}

.finished-card li {
    padding: 8px 0;
    border-bottom: 1px solid #eee;
}

.finished-card li:last-child {
    border-bottom: none;
}

.loading {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(255, 255, 255, 0.8);
    z-index: 999;
}
</style>
