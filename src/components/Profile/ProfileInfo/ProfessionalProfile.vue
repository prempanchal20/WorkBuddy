<template>
    <VForm
        :class="{ editable: !isEdit }"
        :validation-schema="professionalSchema"
        @submit="updateProfessionalInfo"
    >
        <fieldset>
            <legend>Professional Details:</legend>
            <div class="legend-input">
                <label for="qualification">Qualification:</label>
                <VField name="qualification" :bails="false" v-slot="{ errors }">
                    <input
                        list="qualification"
                        id="country"
                        name="qualification"
                        v-model="professional.qualification"
                        :disabled="isEdit"
                        class="input"
                    />
                    <datalist id="qualification">
                        <option value="M.Tech"></option>
                        <option value="B.Tech/B.E"></option>
                        <option value="Diploma"></option>
                        <option value="HSC/SSC"></option>
                    </datalist>
                </VField>
                <div class="error_message" v-for="err in errors" :key="err">
                    {{ err }}
                </div>
            </div>
            <div class="legend-input">
                <label for="cdate">Career Start Date:</label>
                <div>
                    <VField
                        name="cdate"
                        type="date"
                        class="input"
                        v-model="professional.cdate"
                        :disabled="isEdit"
                    />
                    <ErrorMessage name="cdate" class="error_message" />
                </div>
            </div>

            <div class="legend-input">
                <label for="totalExp">Total Experience:</label>
                <div>
                    <VField
                        name="totalExp"
                        type="integer"
                        class="input"
                        disabled
                        v-model="professional.totalExp"
                    />
                    <ErrorMessage name="totalExp" class="error_message" />
                </div>
            </div>

            <div class="button-box">
                <v-slide-group>
                    <v-btn
                        v-if="isEdit"
                        class="ma-2"
                        rounded
                        @click="toggleEdit"
                        id="editBtn"
                    >
                        Edit
                    </v-btn>
                    <template v-else>
                        <v-btn class="ma-2" @click="closeForm" rounded>
                            Close
                        </v-btn>
                        <v-btn class="ma-2" rounded type="submit">
                            Submit
                        </v-btn>
                    </template>
                </v-slide-group>
            </div>
        </fieldset>
    </VForm>
</template>
<script setup lang="ts">
import { reactive, ref, onMounted, watch } from "vue";
import { useFirestore } from "vuefire";
import { Professional } from "@/types/profileTypes";
import { doc, updateDoc, getDoc } from "firebase/firestore";

onMounted(() => {
    priorData();
});

const key = localStorage.getItem("userId");
const db = useFirestore();
const isEdit = ref(true);
const toggleEdit = () => (isEdit.value = !isEdit.value);

const closeForm = () => {
    professional = { ...professionalCopy } as Professional;
    toggleEdit();
};
const professionalSchema = {
    cdate: (value: string) => {
        const inputDate = new Date(value);
        const joiningDate: any = new Date(jdate);
        const dateInFutureError = "Date cannot be ahead of Joining Date";
        if (joiningDate == "Invalid Date") {
            return "Please enter Joining date first";
        } else if (inputDate > joiningDate) {
            return dateInFutureError;
        }
        return true;
    },
};

let jdate: string;
let professional: Professional = reactive({
    qualification: "",
    cdate: "",
    totalExp: "",
});

let professionalCopy: Partial<Professional> = {};
const priorData = async () => {
    const docSnap = await getDoc(doc(db, "users", key));
    if (docSnap.exists()) {
        const employeeData = docSnap.data()?.employee;
        const professionalData = docSnap.data()?.professional;
        jdate = employeeData?.jdate;
        professional.cdate = professionalData?.cdate || "";
        professional.totalExp = professionalData?.totalExp || "";
        professional.qualification = professionalData?.qualification || "";
        professionalCopy = { ...professional };
    } else {
        return;
    }
};

watch(
    () => professional.cdate,
    (newValue) => {
        const diff = Date.now() - new Date(newValue).getTime();
        const years = Math.floor(diff / (1000 * 60 * 60 * 24 * 365));
        const months = Math.floor(diff / (1000 * 60 * 60 * 24 * 30)) % 12;
        professional.totalExp = `${years}Y ${months}M`;
    },
    { deep: true }
);

const updateProfessionalInfo = async (): Promise<void> => {
    await updateDoc(doc(db, "users", key), {
        professional,
    });
    toggleEdit();
};
</script>
<style scoped>
@import "@/assets/profile.css";
</style>
