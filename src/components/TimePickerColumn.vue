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
      <div
        class="item"
        v-for="item in column[type]"
        :key="item"
        :class="{ selected: isSelected(item) }"
      >
        <slot name="item" :item="item"></slot>
        <span v-if="!$slots.item">
          {{ String(item).padStart(2, "0") }}
        </span>
      </div>
    </div>
    <div class="fader"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
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
    : Array.from({ length: 12 }, (_, i) => i)
);
const minutes = computed(() => Array.from({ length: 60 }, (_, i) => i));

const column = ref({
  hours,
  minutes,
});

const ITEM_HEIGHT = 36;
const offset = ref(0);
const startY = ref(0);
const currentY = ref(0);
const velocity = ref(0);
const animationFrame = ref<number>();

const isDragging = ref(false);
const isSelected = (item: number) => {
  const upBorder = (2.3 - item) * ITEM_HEIGHT;
  const downBorder = (1.3 - item) * ITEM_HEIGHT;
  const isInRange = offset.value <= upBorder && offset.value >= downBorder;
  if (isInRange) {
    selectedValue.value = () => item;
  }
  return isInRange;
};

const selectedValue = ref(() => props.selected);

const animate = () => {
  if (Math.abs(velocity.value) < 0.1) {
    velocity.value = 0;
    emit("change", selectedValue.value());
    return;
  }

  offset.value += velocity.value;
  velocity.value *= 0.85;
  animationFrame.value = requestAnimationFrame(animate);
};

const calculateStartValue = () => {
  const index = column.value[props.type].indexOf(props.selected);
  return -(index - 1.5) * ITEM_HEIGHT;
};

function onDragStart(e: TouchEvent | MouseEvent) {
  startY.value = e instanceof MouseEvent ? e.clientY : e.touches[0].clientY;
  isDragging.value = true;
  currentY.value = startY.value;
}

function onDrag(e: TouchEvent | MouseEvent) {
  if (!isDragging.value) return;

  const maxOffset = (column.value[props.type].length - 1) * ITEM_HEIGHT;
  const y = e instanceof MouseEvent ? e.clientY : e.touches[0].clientY;
  const delta = y - currentY.value;

  if (Math.abs(offset.value) >= Math.abs(maxOffset) && delta < 0) {
    return;
  }

  offset.value += delta;
  currentY.value = y;
  velocity.value = delta;
}

function onDragEnd() {
  isDragging.value = false;
  animate();
}

onMounted(() => {
  if (props.selected) {
    offset.value = calculateStartValue();
  } else {
    offset.value = ITEM_HEIGHT * 1.5;
  }
});
</script>

<style scoped lang="scss">
.picker-column {
  height: calc(36px * 4);
  width: 60px;
  overflow: hidden;
  position: relative;
  touch-action: none;

  .picker-column__wrapper {
    position: absolute;
    width: 100%;
    will-change: transform;
    transition: transform 0.3s cubic-bezier(0.23, 1, 0.32, 1);
  }

  .item {
    height: 36px;
    line-height: 36px;
    text-align: center;
    color: #8e8e93;
    transition: all 0.3s;

    &.selected {
      color: #fff;
      font-size: 1.2em;
      font-weight: 500;
    }
  }
}
</style>
