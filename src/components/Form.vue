<template>
  <form @submit.prevent="handleSubmit">
    <label>
      <input
        v-model="formData.title"
        type="text"
        placeholder="event name"
        maxlength="30"
        class="name"
        required
      />
    </label>
    <label class="input-wrapper">
      <span class="input-label" v-if="!formData.selectedDate">event date</span>
      <input
        v-model="formData.selectedDate"
        type="date"
        class="styled-input"
        required
      />
    </label>
    <label class="input-wrapper">
      <span class="input-label" v-if="!formData.time">event time</span>
      <input
        v-model="formData.time"
        type="time"
        class="styled-input"
        required
      />
    </label>
    <label>
      <input
        v-model="formData.note"
        type="text"
        placeholder="notes"
        maxlength="30"
        class="name note"
        required
      />
    </label>
    <div class="buttons">
      <button type="button" @click="$emit('cancel')" class="cancel">
        Cancel
      </button>
      <button type="submit" class="save">Save</button>
    </div>
  </form>
</template>

<script setup lang="ts">
import { ref, watch } from "vue";

const props = defineProps<{
  eventData: {
    title: string;
    selectedDate?: string;
    time?: string;
    note?: string;
  };
}>();

const emit = defineEmits(["save", "cancel"]);
const formData = ref({
  title: "",
  selectedDate: "",
  time: "",
  note: "",
});
watch(
  () => props.eventData,
  (newData) => {
    formData.value.title = newData?.title || "";
    formData.value.selectedDate = newData?.selectedDate || "";
    if (newData?.time) {
      const parsedTime =
        newData.time.length > 5 ? newData.time.slice(0, 5) : newData.time;
      formData.value.time = parsedTime;
    } else {
      formData.value.time = "";
    }
    formData.value.note = newData?.note || "";
  },
  { deep: true, immediate: true }
);
const handleSubmit = () => {
  emit("save", formData.value);
};
</script>

<style scoped>
input {
  font-size: 9px;
  color: #43425d;
  background: transparent;
}
input:focus {
  outline: none;
  background: transparent;
}
.name::placeholder {
  color: #d6d6d6;
}
.name {
  display: block;
  height: 29px;
  font-size: 9px;
  color: #43425d;
  border: none;
  border-bottom: 1px solid #d6d6d6;
  width: 100%;
}
.input-wrapper {
  margin-top: 21px;
  display: flex;
  flex-direction: column;
  font-size: 14px;
  color: #888;
  position: relative;
}
.input-label {
  position: absolute;
  font-size: 9px;
  top: 50%;
  transform: translateY(-50%);
  opacity: 1;
  transition: opacity 0.2s;
  display: block;
  background: white;
  z-index: 9999;
  width: 80%;
  display: block;
  height: 100%;
  color: #d6d6d6;
  font-weight: 400;
}
.input-wrapper input:valid + .input-label {
  opacity: 0;
}
.styled-input {
  display: block;
  height: 28px;
  border: none;
  border-bottom: 1px solid #d6d6d6;
  color: #43425d;
  background: transparent;
  caret-color: inherit;
  z-index: 100;
}
.styled-input:focus {
  outline: none;
  background: transparent;
}
.styled-input::-webkit-calendar-picker-indicator {
  filter: invert(86%) sepia(0%) saturate(0%) hue-rotate(0deg);
  transform: scale(1.3);
  cursor: pointer;
  width: 18px;
  height: 20px;
}
.buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 38px;
}
.note {
  margin-top: 21px;
}
.save {
  border: none;
  background: transparent;
  color: #6a6996;
  cursor: pointer;
  font-size: 12px;
}
.cancel {
  border: none;
  background: transparent;
  color: #ff5f5f;
  cursor: pointer;
  font-size: 12px;
}
.modal-content.editing::after {
  background: rgba(255, 255, 255, 0.5);
  border: 2px solid #6a6996;
}
</style>
