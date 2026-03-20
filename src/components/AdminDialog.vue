<script setup>
import { ref } from "vue";

const nameModel = ref("");
const organizationModel = ref("");
const pathModel = ref("");

const emit = defineEmits(["close-dialog", "post-team"]);
function closeDialog() {
    emit("close-dialog");
}

function postTeam() {
    if (!nameModel.value || !organizationModel.value || !pathModel.value) {
        return;
    }

    const data = {
        name: nameModel.value,
        organization: organizationModel.value,
        path: pathModel.value,
    };
    console.log(data);
    emit("post-team", data);
}

const rules = [
    (value) => {
        if (value) return true;
        return "Field cannot be empty.";
    },
];
</script>

<template>
    <div class="backdrop">
        <v-card class="card">
            <h1>Send team</h1>

            <v-form @submit.prevent="postTeam">
                <v-text-field
                    v-model="nameModel"
                    label="name"
                    :rules
                    required
                ></v-text-field>

                <v-text-field
                    v-model="organizationModel"
                    label="organization"
                    :rules="rules"
                ></v-text-field>

                <v-select
                    v-model="pathModel"
                    label="path"
                    :items="[
                        { title: '123..', value: '1234567890' },
                        { title: '987..', value: '1098765432' },
                    ]"
                    item-title="title"
                    item-value="value"
                    :rules="rules"
                >
                </v-select>

                <v-btn type="submit">Potvrdit</v-btn>
            </v-form>

            <p class="close-text" @click="closeDialog">zavřít</p>
        </v-card>
    </div>
</template>

<style scoped>
.backdrop {
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;

    display: flex;
    justify-content: center;
    align-items: center;

    background-color: rgba(255, 255, 255, 0.8);
}

.card {
    width: 80vw;
    height: 80vh;

    padding: 2rem;

    background-color: #fff;
    opacity: 1;
}

h1 {
    margin-top: 0;
    margin-bottom: 2rem;
}

.close-text {
    cursor: pointer;
    padding: 1rem;

    position: absolute;
    bottom: 2rem;
    right: 2rem;
}
</style>
