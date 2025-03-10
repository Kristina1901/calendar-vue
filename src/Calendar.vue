<template>
  <div class="app-main">
    <p class="app-main-text">Calendar View</p>
    <FullCalendar class="demo-app-calendar" :options="calendarOptions">
      <template v-slot:eventContent="{ event }">
        <div
          :style="{
            color: event.textColor,
            backgroundColor: event.extendedProps.bgColor,
          }"
          class="item"
        >
          <i class="title">{{ event.title }}</i>
          <div class="buttons" @click.stop>
            <button class="delete" @click="deleteEvent(event)"></button>
            <button class="change" @click="changeEventColor(event)"></button>
          </div>
        </div>
      </template>
    </FullCalendar>
    <Modal
      v-if="isModalOpen"
      :title="eventTitle"
      :x="modalX"
      :y="modalY"
      :eventData="formattedEventData"
      @save="handleSaveEvent"
      @close="closeModal"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick, onMounted, onUnmounted } from "vue";
import Modal from "./components/Modal.vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";
import listPlugin from "@fullcalendar/list";
import { INITIAL_EVENTS } from "./event-utils";
import type {
  DateSelectArg,
  EventApi,
  EventClickArg,
} from "@fullcalendar/core";

const selectedInfo = ref<DateSelectArg | null>(null);
const isModalOpen = ref(false);
const eventTitle = ref("");
const currentEvents = ref<EventApi[]>([]);
const modalX = ref(0);
const modalY = ref(0);
const editingEvent = ref<EventApi | null>(null);

const calendarOptions = ref<any>({
  plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin, listPlugin],
  headerToolbar: {
    left: "today,prev,next",
    center: "title",
    right: "dayGridMonth,timeGridWeek,timeGridDay,listWeek",
  },
  buttonText: {
    today: "Today",
    prev: "Back",
    next: "Next",
    dayGridMonth: "Month",
    timeGridWeek: "Week",
    timeGridDay: "Day",
    listWeek: "Agenda",
  },
  initialView: "dayGridMonth",
  initialEvents: INITIAL_EVENTS,
  editable: true,
  selectable: true,
  selectMirror: true,
  dayMaxEvents: true,
  weekends: true,
  select: handleDateSelect,
  eventClick: handleEventClick,
  eventsSet: handleEvents,
  slotEventOverlap: false,
  nowIndicator: true,
  contentHeight: "auto",
  locale: "en",
  views: {
    timeGridDay: {
      titleFormat: { weekday: "long", day: "numeric", month: "short" },
      dayHeaderFormat: {
        weekday: "short",
        day: "2-digit",
        month: "2-digit",
        omitCommas: true,
      },
    },
    timeGridWeek: {
      titleFormat: { day: "numeric", month: "short" },
    },
  },
});
const formattedEventData = computed(() => {
  if (!editingEvent.value) {
    return {
      title: "",
      selectedDate: selectedInfo.value?.startStr || "",
      time: "",
      note: "",
    };
  }
  const eventStart = editingEvent.value.start
    ? new Date(editingEvent.value.start.getTime())
    : null;
  return {
    title: editingEvent.value.title || "",
    selectedDate: eventStart ? eventStart.toISOString().split("T")[0] : "",
    time: eventStart
      ? eventStart.toLocaleTimeString("en-GB", {
          hour: "2-digit",
          minute: "2-digit",
        })
      : "",
    note: editingEvent.value.extendedProps?.note || "",
  };
});
function removeSelectedCellClass() {
  nextTick(() => {
    document.querySelectorAll(".selected-cell").forEach((cell) => {
      cell.classList.remove("selected-cell");
    });
  });
}
function deleteEvent(event: EventApi) {
  if (confirm(`Are you sure you want to delete "${event.title}"?`)) {
    event.remove();
  }
}
function changeEventColor(event: EventApi) {
  const colors = ["#ff5733", "#33ff57", "#3357ff", "#ff33d4", "#f4c542"];
  const randomColor = colors[Math.floor(Math.random() * colors.length)];

  event.setExtendedProp("bgColor", randomColor);
}
function handleDateSelect(selectInfo: any) {
  editingEvent.value = null;
  selectedInfo.value = selectInfo;
  eventTitle.value = "";
  isModalOpen.value = true;
  removeSelectedCellClass();
  const cell = selectInfo.jsEvent.target.closest(".fc-daygrid-day");
  if (cell) {
    cell.classList.add("selected-cell");
  }
  const rect = cell.getBoundingClientRect();
  modalX.value = rect.left + window.scrollX / 2 - 200;
  modalY.value = rect.top + rect.height + window.scrollY - 21;
}
function handleSaveEvent(eventData: {
  title: string;
  selectedDate: string;
  time: string;
  note: string;
}) {
  if (editingEvent.value) {
    editingEvent.value.setProp("title", eventData.title);
    const [hours, minutes] = eventData.time.split(":").map(Number);
    const [year, month, day] = eventData.selectedDate.split("-").map(Number);
    const newStart = new Date(year, month - 1, day, hours, minutes);
    editingEvent.value.setStart(newStart);
    editingEvent.value.setExtendedProp("note", eventData.note);
  } else if (selectedInfo.value) {
    const calendarApi = selectedInfo.value.view?.calendar;
    if (calendarApi) {
      const [hours, minutes] = eventData.time.split(":").map(Number);
      const [year, month, day] = eventData.selectedDate.split("-").map(Number);
      const newStart = new Date(year, month - 1, day, hours, minutes);
      calendarApi.addEvent({
        id: String(Date.now()),
        title: eventData.title.trim(),
        start: newStart,
        allDay: false,
        extendedProps: { note: eventData.note },
      });
    }
  }
  isModalOpen.value = false;
  selectedInfo.value = null;
  editingEvent.value = null;
  removeSelectedCellClass();
}
function closeModal() {
  isModalOpen.value = false;
  selectedInfo.value = null;
  editingEvent.value = null;
  removeSelectedCellClass();
}
function handleEventClick(clickInfo: EventClickArg) {
  editingEvent.value = clickInfo.event;
  eventTitle.value = clickInfo.event.title;
  isModalOpen.value = true;
  removeSelectedCellClass();
  modalX.value = clickInfo.jsEvent.clientX - 460;
  modalY.value = clickInfo.jsEvent.clientY + 20;
  clickInfo.el.classList.add("selected-event");
}
function handleEvents(events: EventApi[]) {
  currentEvents.value = events;
}
onMounted(() => {
  document.addEventListener("click", handleToolbarClick);
});
onUnmounted(() => {
  document.removeEventListener("click", handleToolbarClick);
});
function handleToolbarClick(event: MouseEvent) {
  const toolbarButtons = [
    ".fc-today-button",
    ".fc-prev-button",
    ".fc-next-button",
    ".fc-dayGridMonth-button",
    ".fc-timeGridWeek-button",
    ".fc-timeGridDay-button",
    ".fc-listWeek-button",
  ];
  if (
    toolbarButtons.some((selector) =>
      (event.target as HTMLElement).closest(selector)
    )
  ) {
    closeModal();
  }
}
</script>
<style lang="css">
.app-main {
  padding: 20px;
}
table {
  width: 100%;
}
.fc {
  max-width: 1130px;
  margin: 0 auto;
}
.fc-toolbar-title {
  color: #4d4f5c;
  font-size: 18px;
}
.fc-button-group .fc-button {
  border: 1px solid #d7dae2;
  border-radius: 4px 0px 0px 4px;
  background: transparent;
  color: #4d4f5c;
  font-size: 13px;
}
.fc-button-group .fc-button:hover {
  border: 1px solid #d7dae2;
  border-radius: 4px 0px 0px 4px;
  background: transparent;
  color: #4d4f5c;
}
.fc-button-active:focus {
  border: none;
  box-shadow: none;
}
.fc-button-active:hover {
  border: none;
  box-shadow: none;
}
.fc-button-active {
  border: 1px solid #d7dae2;
  border-radius: 4px 0px 0px 4px;
  background: transparent;
  color: #4d4f5c;
}
.fc-button-group .fc-button:disabled {
  background: transparent;
  color: #3b86ff;
  border: 1px solid #d7dae2;
}
.fc-button:last-child {
  border-radius: 4px;
}

.fc .fc-button-primary:focus {
  box-shadow: none;
}
.fc-button-group .fc-button:first-child {
  width: 69px;
}
.fc-button-group .fc-button:nth-child(2) {
  width: 58px;
}
.fc-button-group .fc-button:nth-child(3) {
  width: 68px;
}
.fc-agenda-button {
  position: relative;
  width: 78px;
}
.fc-agenda-button::before {
  content: "Agenda";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.fc-toolbar-chunk .fc-button-group {
  box-shadow: none !important;
}
.fc-toolbar-chunk .fc-button-group .fc-button-active {
  color: #3b86ff !important;
  background: transparent !important;
  border: 1px solid #d7dae2 !important;
}
thead tr .fc-scrollgrid-sync-inner {
  width: 100%;
  height: 100%;
  align-items: center;
  display: flex;
  justify-content: center;
}

thead tr {
  height: 45px;
  border: 1px solid #eaf0f4;
  background: #f5f6fa;
}
.fc-daygrid table thead tr th {
  width: 162px;
}
.fc-col-header-cell-cushion {
  font-size: 14px;
  color: #a3a6b4;
}
.fc-daygrid-day-number {
  font-size: 15px;
  color: #43425d;
}
.fc-daygrid-day-top {
  padding-top: 15px;
  padding-right: 15px;
}
.fc .fc-highlight {
  background: #eaf0f4;
}
.fc-daygrid table tbody .fc-day {
  width: 162px;
  height: 135px;
}
.fc .fc-daygrid-day .fc-day-today {
  background: #f5f6fa;
}
.fc-timeGridWeek-view table .fc-daygrid-day {
  width: 150px;
  height: 49px;
}
.fc-timeGridWeek-view table thead .fc-timegrid-axis {
  width: 80px;
  border-right: none !important;
}
.fc-timeGridWeek-view table thead th {
  width: 150px;
  height: 49px;
}
.fc-timeGridWeek-view table .fc-timegrid-slot-minor {
  border: none;
}
.custom-now-indicator {
  height: 2px;
  border-top: 2px solid #3b86ff !important;
  width: 100%;
}
::v-deep .fc-timegrid-axis {
  width: 80px !important;
}
.fc-timegrid-divider {
  display: none !important;
}
.fc .fc-timegrid-axis-frame {
  justify-content: center;
}
.fc-day-sun {
  border-left: none !important;
}
.fc-timeGridWeek-view .fc-day-sun .fc-scrollgrid-sync-inner {
  justify-content: start;
}
.fc-timegrid-slot-label {
  width: 80px;
}
.fc-slats td {
  height: 10px !important;
  font-size: 8px !important;
}
.fc-timeGridWeek-view .fc-day-today {
  --fc-today-bg-color: #f5f6fa;
}
.fc-dayGridMonth-view .fc-day-today {
  --fc-today-bg-color: #f5f6fa;
}
.fc-timeGridDay-view .fc-day-today {
  --fc-today-bg-color: #f5f6fa;
}
.fc-timegrid-slot-lane {
  min-width: 80px;
}
.fc-timeGridWeek-view table tr:nth-child(even) {
  display: none;
}
.fc-timeGridDay-view table tr:nth-child(even) {
  display: none;
}
.fc-timeGridDay-view .fc-timegrid-slots tr {
  height: 49px;
}
.fc-timeGridDay-view .fc-daygrid-day-frame .fc-daygrid-day-events a {
  height: 49px;
  padding-top: 15px;
  padding-left: 16px;
  text-align: center;
  border: 1px solid #eaf0f4;
}
.fc .fc-daygrid-body-natural .fc-daygrid-day-events {
  margin-bottom: 1px;
}
.fc-timeGridWeek-view .fc-timegrid-slots tr {
  height: 49px;
}
.fc-timeGridWeek-view .fc-daygrid-event-harness a {
  height: 23px;
  padding-top: 3px;
  padding-bottom: 3px;
}
.fc-timegrid-slot-label-cushion {
  color: #4d4f5c;
  font-size: 13px;
}
.fc-timegrid-axis-cushion {
  color: #4d4f5c;
  font-size: 13px;
}
.fc-direction-ltr .fc-timegrid-slot-label-frame {
  text-align: center;
}
.fc-timeGridWeek-view .fc-event-main b {
  display: none;
}
.fc-timeGridWeek-view .fc-event-main i {
  color: #ffffff !important;
  font-size: 13px;
}
.custom-now-indicator {
  position: relative;
  z-index: 10;
  height: 100%;
  background-color: #3b86ff;
  height: 2px;
  width: 100%;
  border-radius: 2px;
  transition: all 0.5s ease;
}
.custom-now-indicator::before {
  content: "";
  background: #3b86ff;
  width: 8px;
  z-index: 10;
  height: 8px;
  display: block;
  border-radius: 50%;
  position: absolute;
  left: -0;
  top: -5px;
}
.fc .fc-timegrid-now-indicator-arrow {
  display: none;
}
.app-main {
  max-width: 1130px;
  margin: 0 auto;
  position: relative;
}
.app-main-text {
  font-size: 18px;
  color: #4d4f5c;
}
.fc-toolbar-title {
  font-weight: 400;
  margin-bottom: 15px;
}
.fc-toolbar-chunk:last-child .fc-button-group {
  position: relative;
  top: -45px;
}
.fc-direction-ltr .fc-daygrid-event.fc-event-end {
  margin: 0;
  margin-bottom: 2px;
}
.selected-cell .fc-highlight {
  background-color: transparent !important;
  box-shadow: 0px 3px 6px #00000029;
}
.selected-cell .fc-daygrid-day-frame .fc-daygrid-day-top a {
  color: #43425d;
  text-shadow: 2px 2px 4px rgba(67, 66, 93, 0.5);
}
.fc-daygrid-day-events {
  overflow: hidden;
}
.selected-event {
  background-color: transparent !important;
  border: 1px solid #3b86ff;
  color: #3b86ff !important;
}
.selected-event .fc-event-main i {
  color: #3b86ff !important;
}
.fc-daygrid-dot-event {
  background-color: #3b86ff;
  font-size: 13px;
  color: white;
  padding: 0;
  border-radius: 4px;
}
.delete {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #d6d6d6;
  border-radius: 50%;
  background-color: transparent;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23ffffff' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'><line x1='18' y1='6' x2='6' y2='18'></line><line x1='6' y1='6' x2='18' y2='18'></line></svg>");
  background-repeat: no-repeat;
  background-position: center;
  cursor: pointer;
  transition: all 0.3s ease;
  position: absolute;
  right: 6px;
  top: 6px;
}
.change {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #d6d6d6;
  border-radius: 50%;
  background-color: transparent;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23ffffff' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'><path d='M23 4v6h-6'></path><path d='M1 20v-6h6'></path><path d='M3.51 9a9 9 0 0 1 14.38-3L23 10'></path><path d='M20.49 15a9 9 0 0 1-14.38 3L1 14'></path></svg>");
  background-repeat: no-repeat;
  background-position: center;
  cursor: pointer;
  transition: all 0.3s ease;
  position: absolute;
  right: 36px;
  top: 6px;
}
.title {
  width: 80px;
  overflow: hidden;
}
.selected-event i {
  color: #3b86ff !important;
}
.item {
  width: 100%;
  height: 100%;
  padding: 7px 14px;
  border-radius: 4px;
}
</style>
