<template>
  <main class="main">
    <div class="time-picker" v-if="!modalIsOpened" @click.stop="openModal">
      <h1 class="time-picker__title">{{ title }}</h1>
      <div class="time-picker__main">
        <span>{{ modelValue || defaultTime }}</span>
        &nbsp;
        <span class="time-picker__format">{{ appm }}</span>
      </div>
    </div>
    <transition name="modal">
      <div class="time-picker__modal" v-if="modalIsOpened">
        <div class="modal-wrapper" ref="modalWrapperRef">
          <div class="picker-columns">
            <TimePickerColumn
              type="hours"
              :format="timeFormat"
              @change="setHours"
              :selected="hours"
            >
              <template #item="item">
                <span
                  :class="{ disabled: isMoreThanMaxtime || isLessThanMintime }"
                >
                  {{ String(item.item).padStart(2, "0") }}
                </span>
              </template>
            </TimePickerColumn>
            <TimePickerColumn
              type="minutes"
              @change="setMinutes"
              :selected="minutes"
            >
              <template #item="item">
                <span
                  :class="{ disabled: isMoreThanMaxtime || isLessThanMintime }"
                >
                  {{ String(item.item).padStart(2, "0") }}
                </span>
              </template></TimePickerColumn
            >
            <span class="modal__format" @click="toggleTimeFormat"
              >{{ appm }}
            </span>
          </div>
          <div
            class="close-modal-button"
            :class="{
              'disabled-button': isMoreThanMaxtime || isLessThanMintime,
            }"
            @click="() => closeModal(true)"
          >
            Confirm
          </div>
        </div>
      </div>
    </transition>
  </main>
</template>

<script setup lang="ts">
import { ref, computed, watch, onBeforeUnmount } from "vue";
import TimePickerColumn from "./TimePickerColumn.vue";
import type { ITimeFormat } from "../types";

interface IProps {
  modelValue: string;
  title?: string;
  format?: ITimeFormat;
  minTime?: string;
  maxTime?: string;
}

const props = withDefaults(defineProps<IProps>(), {
  modelValue: "",
  title: "Time Picker",
  format: "24h",
  minTime: "02:10",
  maxTime: "19:30",
});

const emit = defineEmits(["update:modelValue"]);

const defaultTime = ref("--:--");
const timeFormat = ref<ITimeFormat>("24h");

const toggleTimeFormat = () => {
  timeFormat.value = timeFormat.value === "12h" ? "24h" : "12h";
};

const appm = computed(() => {
  if (hours.value < 12) return "AM";
  if (hours.value >= 12) return "PM";
  return null;
});

const hours = ref<number>(0);
const minutes = ref<number>(0);

const setHours = (value: number) => {
  hours.value = value;
};
const setMinutes = (value: number) => {
  minutes.value = value;
};

const isMoreThanMaxtime = computed(() => {
  const maxTime = props.maxTime.split(":");
  const maxHours = Number(maxTime[0]);
  const maxMinutes = Number(maxTime[1]);
  return (
    hours.value > maxHours ||
    (hours.value > maxHours - 1 && minutes.value > maxMinutes)
  );
});

const isLessThanMintime = computed(() => {
  const minTime = props.minTime.split(":");
  const minHours = Number(minTime[0]);
  const minMinutes = Number(minTime[1]);
  return (
    hours.value < minHours ||
    (hours.value < minHours + 1 && minutes.value < minMinutes)
  );
});

const currentTimepickerTime = computed(
  () =>
    `${String(hours.value).padStart(2, "0")}:${String(minutes.value).padStart(
      2,
      "0"
    )}`
);

const modalIsOpened = ref(false);
const modalWrapperRef = ref<HTMLElement | null>(null);
const handleClickOutside = (event: MouseEvent) => {
  if (
    modalWrapperRef.value &&
    !modalWrapperRef.value.contains(event.target as Node)
  ) {
    closeModal(false);
  }
};

const closeModal = (withSave: boolean) => {
  modalIsOpened.value = false;
  if (withSave) {
    emit("update:modelValue", currentTimepickerTime.value);
  }
};

const openModal = () => {
  modalIsOpened.value = true;
  if (!props.modelValue) {
    const currentDay = new Date();
    setHours(currentDay.getHours());
    setMinutes(currentDay.getMinutes());
  }
};

watch(modalIsOpened, (newValue) => {
  if (newValue) {
    document.addEventListener("click", handleClickOutside);
  } else {
    document.removeEventListener("click", handleClickOutside);
  }
});

onBeforeUnmount(() => {
  document.removeEventListener("click", handleClickOutside);
});
</script>

<style scoped lang="scss">
.main {
  width: 100%;
}

.time-picker {
  background-color: var(--tg-theme-text-color);
  display: flex;
  justify-content: space-between;
  border-radius: 8px;

  padding: 0 1rem;

  .time-picker__title {
    color: var(--tg-theme-header-bg-color);
  }
}

.time-picker__main {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px;
  border-radius: 8px;
  background: var(--tg-theme-text-color);
  color: var(--tg-theme-header-bg-color);
  cursor: pointer;

  font-size: 1.3rem;

  .time-picker__format {
    font-weight: 700;
  }
}

.time-picker__modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: flex-end;
  flex-direction: column;

  .modal__format {
    margin-top: 6px;
    font-size: 1.2rem;
    color: #fff;
    align-self: center;
  }

  .modal-wrapper {
    background: var(--tg-theme-text-color);
    height: 200px;
    padding: 16px;
    border-radius: 12px 12px 0 0;

    .picker-columns {
      display: flex;
      justify-content: center;
      gap: 8px;
      height: 140px;
    }

    .close-modal-button {
      width: 100%;
      padding: 12px;
      background: var(--button-color);
      color: white;
      border: none;
      border-radius: 8px;
      margin-top: 8px;
      text-align: center;
    }
  }
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s;

  .modal {
    transition: transform 0.3s;
  }
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;

  .modal {
    transform: translateY(100%);
  }
}

.disabled {
  color: #757575;
}

.disabled-button {
  color: #757575 !important;
  cursor: default !important;
  pointer-events: none !important;
}
</style>
