<script setup>
const { teams } = defineProps(["teams"]);

const statusClass = (team) => {
    if (team.path == "") {
        return "finished";
    } else if (team.logs[0]) {
        return team.logs[0].status;
    } else {
        return "";
    }
};
</script>

<template>
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
                <td>{{ team.logs[0] ? team.logs[0].name : "-" }}</td>
                <td>
                    <span class="status" :class="statusClass(team)">
                        {{
                            team.logs[0]
                                ? team.logs[0].status +
                                  " " +
                                  new Date(
                                      team.logs[0].created_at + "Z",
                                  ).toLocaleTimeString()
                                : "-"
                        }}
                    </span>
                </td>
            </tr>
        </tbody>
    </v-table>
</template>

<style scoped>
th {
    font-weight: bold !important;
}

.status {
    padding: 0.6rem;
    border-radius: 50px;
}

.departed {
    background-color: #06a77d;
    color: #03402f;
}

.arrived {
    background-color: #367ac4;
    color: #122c49;
}

.finished {
    background-color: #cccccc;
    color: #000;
}
</style>
