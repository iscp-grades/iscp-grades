<template>
  <v-container fluid class="d-flex flex-column align-center">
    <v-dialog v-model="loading" hide-overlay persistent>
      <v-progress-circular
        indeterminate
        color="white"
        class="mb-0"
      ></v-progress-circular>
    </v-dialog>

    <template v-if="!submitted">
      <v-img :src="iscpLogoSrc" width="320px"></v-img>
      <span class="text-sm-h4 text-md-h4 text-lg-h4 text-xl-h4 my-1"
        >ISCP v0.34</span
      >
      <v-form @submit.prevent="submitStudentNumber">
        <v-text-field
          required
          autofocus
          label="Student Number"
          placeholder="e.g. 0034985676"
          type="number"
          :error="invalidInput"
          v-model="studentNumber"
          class="inputs"
          hide-details
        ></v-text-field>
        <v-btn
          class="mt-2 inputs"
          @click="submitStudentNumber"
          :disabled="!isValidStudentNumber"
          >Submit</v-btn
        >
      </v-form>
    </template>
    <template v-else>
      <v-container fluid class="grades-table" v-if="!loading">
        <v-table density="compact">
          <caption class="">
            <v-container
              class="ma-0 pa-0 fluid d-flex flex-row justify-space-between"
            >
              <span class="text-left text-uppercase font-weight-bold"
                >Student Number: {{ `${currentYear}-${studentNumber}` }}</span
              >
              <v-btn variant="outlined" color="blue" @click="reset"
                >Go Back</v-btn
              >
              <!-- <v-btn icon="mdi-keyboard-backspace" color="secondary" flat></v-btn> -->
            </v-container>
          </caption>
          <thead>
            <tr>
              <th class="text-uppercase text-center">Subject</th>
              <th class="text-uppercase text-center">Grade</th>
              <th class="text-uppercase text-center">Comment</th>
            </tr>
          </thead>
          <tbody>
            <tr :key="item.subject + '-' + key" v-for="(item, key) in grades">
              <td>{{ item.subject }}</td>
              <td class="text-right font-weight-bold">
                {{
                  item.grade.toLocaleString(undefined, {
                    maximumFractionDigits: 1,
                    minimumFractionDigits: 1,
                  })
                }}
              </td>
              <td class="text-right font-weight-bold">
                <span v-if="item.comment === 'PASSED'" class="text-green">{{
                  item.comment
                }}</span>
                <span v-else class="text-red">{{ item.comment }}</span>
              </td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td class="text-right font-weight-bold text-right">TOTAL:</td>
              <td class="text-right font-weight-bold">{{ averageGrade }}</td>
              <td class="text-right font-weight-bold">
                <span
                  v-if="averageGrade <= 3"
                  class="text-green font-weight-bold"
                  >PASSED</span
                >

                <span v-else class="text-red font-weight-bold">FAILED</span>
              </td>
            </tr>
          </tfoot>
        </v-table>
      </v-container>
    </template>

    <!-- Footer -->
    <v-container fluid class="text-center" v-if="!loading">
      <span>© 2022 ISCP Andromeda Galaxy IT Dept.</span>
    </v-container>
  </v-container>
</template>

<script setup>
import { ref, onMounted, computed, reactive } from "vue";
import CryptoJS from "crypto-js";

// Variables
const iscpLogoSrc = ref("https://i.imgur.com/ucqq9ho.png");
let studentNumber = ref(null);
let errorDialog = ref(false);
let loading = ref(false);
let invalidInput = ref(false);
const submitted = ref(false);
const currentYear = ref(new Date().getFullYear());
let averageGrade = ref(0);

let subjects = ref([
  "BS223 Virtual Streaming",
  "BS069 Tinapay",
  "BS984 Adobe Valorant",
  "BA456 English written in Filipino",
  "CS101 Pisonet Structures and Algorithms",
  "CSM104 Introduction to Pancit Canton Cooking",
  "BA394 Bold Searching",
  "BA542 Japanese in Filipino",
  "BA564 Filipino in English",
  "BA343 Math 305 - Elementary Addition",
  "BA344 Math 306 - Elementary Subtraction",
  "BA345 Math 307 - Elementary Multiplication",
  "BA346 Math 308 - Elementary Division",
  "BS983 Witchcraft",
  "BS345 Airbending",
  "BS984 Advanced Bloodbending",
  "BS345 AFAM Hunting",
]);
let grades = reactive([]);

// Computed
const isValidStudentNumber = computed(() => {
  return /^\d+$/.test(studentNumber.value);
});

// Functions
const isNumeric = (num) => {
  return !isNaN(num);
};

const startFakeLoading = () => {
  loading.value = true;
  const randomDuration = Math.floor(Math.random() * 2000);

  setTimeout(() => {
    loading.value = false;
    computeGrades();
  }, randomDuration);
};

// Reset
const reset = () => {
  studentNumber.value = null;
  grades.splice(0, grades.length);
  submitted.value = false;
};

const computeGrades = () => {
  // SHA256 because it's long
  const sha256 = CryptoJS.SHA256(studentNumber.value).toString();

  let numSubjects = subjects.value.length;
  let char = null;
  let charValue = null;
  let grade = 0;
  let average = 0;
  const offset = 7; // kasi masyado maraming pasado

  for (let i = 0; i < numSubjects; i++) {
    char = sha256[i];

    // If not number, convert to upper case
    if (!isNumeric(char)) {
      char = char.toUpperCase();
      charValue = char.charCodeAt(0);
      grade = ((charValue - 65) / 26) * 100;
    } else {
      charValue = parseInt(char) + offset;
      grade = (charValue / 26) * 100;
    }

    let convertedGrade = (5 - grade / numSubjects).toFixed(1);
    convertedGrade =
      convertedGrade >= 0 && convertedGrade < 1 ? 1.0 : convertedGrade; // Kapag between 0 and 1, uno;
    convertedGrade =
      convertedGrade > 3.0 ? Math.ceil(convertedGrade) : convertedGrade; // Kapag higher than tres, round up, ibagsak

    average += parseFloat(convertedGrade);

    let comment = "";
    if (convertedGrade <= 3.0) {
      comment = "PASSED";
    } else if (convertedGrade === 4) {
      comment = "DROPPED";
    } else {
      comment = "FAILED";
    }
    grades.push({
      subject: subjects.value[i],
      grade: convertedGrade,
      comment: comment,
    });
  }

  // Check if greater than 3, pag oo, ibagsak
  const ibagsakBa = average / numSubjects;
  if (ibagsakBa > 3) {
    average = Math.ceil(ibagsakBa); // Ibagsak
  }

  averageGrade.value = average.toFixed(1);
};

const submitStudentNumber = () => {
  if (!isValidStudentNumber.value) {
    // TODO: Show error message
    errorDialog.value = true;
    invalidInput.value = true;
    return;
  }
  invalidInput.value = false;
  submitted.value = true;

  startFakeLoading();
};
</script>

<style scoped>
.inputs {
  width: 320px;
}

.grades-table {
  border: 1px solid #ccc;
  width: 800px;
}
</style>