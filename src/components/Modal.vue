<template>
  <div class="modal" :style="modalStyle">
    <button class="close" @click="$emit('close')"></button>
    <div class="modal-content" :class="{ editing: isEditing }">
      <Form :eventData="eventData" @save="saveEvent" @cancel="$emit('close')" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
import Form from "./Form.vue";

const props = defineProps<{
  eventData: {
    title: string;
    date?: string;
    time?: string;
    notes?: string;
  };
  x: number;
  y: number;
}>();
const emit = defineEmits(["save", "close"]);
const saveEvent = (eventData: {
  title: string;
  date: string;
  time: string;
  notes: string;
}) => {
  emit("save", eventData);
};
const modalStyle = computed(() => ({
  top: `${props.y}px`,
  left: `${props.x}px`,
}));
const isEditing = computed(() => !!props.eventData.title);
</script>
<style scoped>
.modal {
  position: absolute;
  background: white;
  border-radius: 8px;
  z-index: 1000;
  box-shadow: 0px 3px 18px #00000029;
  border: 1px solid #43425d;
  border-radius: 10px;
  padding-top: 26px;
  width: 201px;
  height: 275px;
}
.close {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #d6d6d6;
  border-radius: 50%;
  background-color: transparent;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23d6d6d6' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'><line x1='18' y1='6' x2='6' y2='18'></line><line x1='6' y1='6' x2='18' y2='18'></line></svg>");
  background-repeat: no-repeat;
  background-position: center;
  cursor: pointer;
  transition: all 0.3s ease;
  position: absolute;
  right: 6px;
  top: 6px;
}
.modal-content {
  position: relative;
  width: 148px;
  margin: 0 auto;
}
.modal-content::after {
  content: "";
  position: absolute;
  top: -39px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 9px solid transparent;
  border-right: 9px solid transparent;
  border-bottom: 13px solid #43425d;
}
.editing:before {
  content: "";
  position: absolute;
  top: -41px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 8px solid transparent;
  border-right: 8px solid transparent;
  border-bottom: 14px solid #43425d;
}
.editing::after {
  content: "";
  position: absolute;
  top: -39px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 7px solid transparent;
  border-right: 7px solid transparent;
  border-bottom: 12px solid #f5f6fa;
}
</style>
