<template>
    <div
        @mousedown.passive="onDragStart"
        @mousemove.passive="onDrag"
        @mouseup.passive="onDragEnd"
        @mouseleave.passive="onDragEnd"
        @touchstart.passive="onDragStart"
        @touchmove.passive="onDrag"
        @touchend.passive="onDragEnd"
        class="picker-column"
    >
        <div
            class="picker-column__wrapper"
            :style="{ transform: `translateY(${offset}px)` }"
        >
            <div class="item" v-for="item in column[type]" :key="item">
                {{ item }}
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";
import type { ITimeFormat } from "../types";

interface IProps {
    type?: "hours" | "minutes";
    selected?: number;
    format?: ITimeFormat;
}

const props = withDefaults(defineProps<IProps>(), {
    type: "hours",
    selected: 0,
    format: "24h",
});

const emit = defineEmits(["change"]);

const hours = computed(() =>
    props.format === "24h"
        ? Array.from({ length: 24 }, (_, i) => i)
        : Array.from({ length: 12 }, (_, i) => ++i),
);
const minutes = computed(() => Array.from({ length: 60 }, (_, i) => i));

const column = ref({
    hours,
    minutes,
});

const itemHeight = 36;
const offset = ref(0);
const startY = ref(0);
const currentY = ref(0);
const velocity = ref(0);

const isDragging = ref(false);

const calculateStartValue = () => {
    const index = column.value[props.type].indexOf(props.selected);
    return -index * itemHeight;
};

function onDragStart(e: TouchEvent | MouseEvent) {
    startY.value = e instanceof MouseEvent ? e.clientY : e.touches[0].clientY;
    isDragging.value = true;
    currentY.value = startY.value;
}

function onDrag(e: TouchEvent | MouseEvent) {
    if (!isDragging.value) return;

    const y = e instanceof MouseEvent ? e.clientY : e.touches[0].clientY;
    const delta = y - currentY.value;
    offset.value += delta;
    currentY.value = y;
    velocity.value = delta;
}

function onDragEnd() {
    isDragging.value = false;
    const index = Math.round(-offset.value / itemHeight);
    const clamped = Math.max(
        0,
        Math.min(index, column.value[props.type].length - 1),
    );
    offset.value = -clamped * itemHeight;
    const selectedItem = column.value[props.type][clamped];

    emit("change", selectedItem);
}

onMounted(() => {
    console.log("Props,", props.selected);
    if (props.selected) {
        offset.value = calculateStartValue();
    }
});
</script>

<style scoped lang="scss">
.picker-column {
    height: 100px;
    height: 100px;
    width: 2rem;
    overflow: hidden;
    position: relative;

    .picker-column__wrapper {
        position: absolute;
        transition: transform 0.3s cubic-bezier(0.23, 1, 0.32, 1);
    }

    .item {
        height: 36px;
        line-height: 36px;
        text-align: center;
        color: var(--tg-theme-hint-color);
        transition: all 0.3s;
    }
}
</style>
