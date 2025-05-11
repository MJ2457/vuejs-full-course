<script setup>
import Welcome from "./components/pages/Welcome.vue";
import Layout from "./components/layouts/Layout.vue";
import Dashboard from "./components/pages/Dashboard.vue";
import Workout from "./components/pages/Workout.vue";
import { ref, computed, onMounted } from "vue";
import { workoutProgram } from "./utils";

const defaultData = {};
for (let workoutIdx in workoutProgram) {
  const workoutData = workoutProgram[workoutIdx];
  defaultData[workoutIdx] = {};
  for (let e of workoutData.workout) {
    defaultData[workoutIdx][e.name] = "";
  }
}

const selectedDisplay = ref(1); // 1 = Welcome, 2 = Dashboard, 3 = Workout
const data = ref(defaultData); // {1....30: {exersize_name: {sets: {1: {reps: 10, weight: 20}}}}}
const selectedWorkout = ref(-1);

const isWorkoutComplete = computed(() => {
  const currWorkout = data.value?.[selectedWorkout.value];
  if (!currWorkout) {
    return false;
  } // guard clause to exit function if no workout is selected

  const isCompleteCheck = Object.values(currWorkout).every((exercise) => !!exercise);
  console.log("ISCOMPLETE ", isCompleteCheck);
  return isCompleteCheck;
});

const firstIncompleteWorkoutIndex = computed(() => {
  const allWorkouts = data.value;
  if (!allWorkouts) {
    return -1;
  } // guard clause to exit function if no workouts are selected

  // loop over every key value pair, and check if the workout is complete or not.
  for (const [index, workout] of Object.entries(allWorkouts)) {
    const isComplete = Object.values(workout).every((exercise) => !!exercise);
    if (!isComplete) {
      return parseInt(index);
    }
  }
  return -1; //all are complete
});

function handleChangeDisplay(idx) {
  selectedDisplay.value = idx;
}

function handleSelectWorkout(idx) {
  selectedDisplay.value = 3;
  selectedWorkout.value = idx;
}

function handleSaveWorkout() {
  // Save the workout data to local storage or a database
  localStorage.setItem("workouts", JSON.stringify(data.value));
  // show the dashboard
  selectedDisplay.value = 2;
  // deselect a workout
  selectedWorkout.value = -1;
}

function handleResetPlan() {
  // Reset the workout data to default
  data.value = defaultData;
  // show the dashboard
  selectedDisplay.value = 2;
  // deselect a workout
  selectedWorkout.value = -1;
  localStorage.removeItem("workouts");
}

onMounted(() => {
  if (!localStorage) {
    return;
  }
  if (localStorage.getItem("workouts")) {
    const savedData = JSON.parse(localStorage.getItem("workouts"));
    data.value = savedData;
    selectedDisplay.value = 2;
  }
});
</script>

<template>
  <Layout>
    <!-- Page 1 -->
    <Welcome :handleChangeDisplay="handleChangeDisplay" v-if="selectedDisplay == 1" />
    <!-- Page 2 -->
    <Dashboard
      :handleResetPLan="handleResetPLan"
      :firstIncompleteWorkoutIndex="firstIncompleteWorkoutIndex"
      :handleSelectWorkout="handleSelectWorkout"
      v-if="selectedDisplay == 2"
    />
    <!-- Page 3 -->
    <Workout
      :isWorkoutComplete="isWorkoutComplete"
      :handleSaveWorkout="handleSaveWorkout"
      :data="data"
      :selectedWorkout="selectedWorkout"
      v-if="workoutProgram?.[selectedWorkout]"
    />
  </Layout>
</template>

<style scoped></style>
