<script setup lang="ts">
import { ref } from "vue";
import TimePicker from "./components/TimePicker.vue";

const time = ref('')
</script>

<template>
  <TimePicker v-model="time"/>
</template>
