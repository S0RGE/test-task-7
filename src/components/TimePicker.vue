<template>
    <main>
        <h1>{{ title }}</h1>
        <div class="time-picker" v-if="!modalIsOpened">
            <input type="checkbox" />
            <div>
                <span @click="toggleModal">{{
                    modelValue || defaultTime
                }}</span>
                &nbsp;
                <span class="time-picker__format">{{ appm }}</span>
            </div>
        </div>
        <div class="time-picker__modal" v-if="modalIsOpened">
            <div class="modal-wrapper">
                <TimePickerColumn
                    type="hours"
                    :format="timeFormat"
                    @change="setHours"
                    :selected="hours"
                />
                <TimePickerColumn
                    type="minutes"
                    @change="setMinutes"
                    :selected="minutes"
                />
                <span v-if="timeFormat === '12h'" @click="toggleTimeFormat">{{
                    appm
                }}</span>
            </div>
            <div class="close-modal-button" @click="toggleModal">Close</div>
        </div>
    </main>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";
import TimePickerColumn from "./TimePickerColumn.vue";
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
    title: "Time Picker",
    format: "24h",
    minTime: "00:00",
    maxTime: "23:59",
});

const emit = defineEmits(["update:modelValue"]);

const defaultTime = ref("--:--");
const timeFormat = ref<ITimeFormat>("12h");

const toggleTimeFormat = () => {
    timeFormat.value = timeFormat.value === "12h" ? "24h" : "12h";
};

const appm = computed(() => {
    if (timeFormat.value === "12h" && hours.value < 12) return "AM";
    if (timeFormat.value === "12h" && hours.value > 12) return "PM";
    return null;
});

const hours = ref<number>(0); //TODO: fix this
const minutes = ref<number>(0);
const setHours = (value: number) => {
    hours.value = value;
};
const setMinutes = (value: number) => {
    minutes.value = value;
};

const modalIsOpened = ref(false);
const toggleModal = () => {
    modalIsOpened.value = !modalIsOpened.value;
};

watch(modalIsOpened, (newValue) => {
    if (!newValue) {
        const time = `${String(hours.value).padStart(2, "0")}:${String(minutes.value).padStart(2, "0")}`;
        emit("update:modelValue", time);
    }
});

watch([hours, minutes], () => {
    if (
        hours.value < 0 ||
        hours.value > 23 ||
        minutes.value < 0 ||
        minutes.value > 59
    ) {
        hours.value = 0;
        minutes.value = 0;
    }
});
</script>

<style scoped lang="scss">
.time-picker {
    display: flex;
    flex-direction: column;
}

.modal-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
}
</style>
