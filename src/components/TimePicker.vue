<template>
  <div>
    <h1>TimePicker</h1>
  </div>
</template>

<script setup lang="ts">
import type { ITimeFormat } from "../types";

interface IProps {
  modelValue: string;
  title?: string;
  format?: ITimeFormat;
  minTime?: string;
  maxTime?: string;
}

withDefaults(defineProps<IProps>(), {
  modelValue: "",
  title: "Time",
  format: "24h",
  minTime: "00:00",
  maxTime: "23:59",
});

defineEmits(["update:modelValue"]);
</script>

<style scoped lang="scss"></style>
