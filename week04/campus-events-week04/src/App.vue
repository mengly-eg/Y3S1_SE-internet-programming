<script setup lang="ts">
import { computed, ref, watch } from "vue";
import EventList from "./components/EventList.vue";
import { initialEvents } from "./data/events";
import type { CampusEvent } from "./types";

// A reactive copy of the hard-coded events
const events = ref<CampusEvent[]>(initialEvents.map(e => ({ ...e })));

// TODO 1: a ref `myBookings` holding the ids of the events I booked (one entry per seat)
const myBookings = ref<number[]>([]);

// TODO 2: a computed `bookingCount`
const bookingCount = computed(() => myBookings.value.length);

// TODO 3: onBook(id): find the event; if it still has a seat,
//         increase its `booked` and add the id to myBookings
function onBook(id: number) {
  const event = events.value.find(e => e.id === id);
  if (event && event.booked < event.capacity) {
    event.booked++;
    myBookings.value.push(id);
  }
}

// TODO 4 (bonus): keep my bookings after a page reload
//   - watch myBookings and save it to localStorage (JSON.stringify)
//   - read it back when the app starts, and add those seats to `booked`
try {
  const stored = localStorage.getItem("myBookings");
  if (stored) {
    const ids: number[] = JSON.parse(stored);
    myBookings.value = ids;
    for (const id of ids) {
      const event = events.value.find(e => e.id === id);
      if (event && event.booked < event.capacity) {
        event.booked++;
      }
    }
  }
} catch {
  // ignore invalid JSON
}

// Save bookings whenever they change
watch(myBookings, (val) => {
  localStorage.setItem("myBookings", JSON.stringify(val));
}, { deep: true });
</script>

<template>
  <main>
    <header>
      <h1>Campus Events</h1>
      <!-- Show "1 seat" or "N seats" -->
      <p class="mine">
        You have booked {{ bookingCount }} {{ bookingCount === 1 ? "seat" : "seats" }}.
      </p>
    </header>
    <EventList :events="events" @book="onBook" />
  </main>
</template>

<style scoped>
header { margin-bottom: 16px; }
.mine { color: #6b7280; margin: 0; }
</style>
