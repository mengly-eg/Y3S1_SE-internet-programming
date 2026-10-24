# Week 4 – Explain It

## 1. What goes wrong if you use the array index as `key` in `EventList` when the list is filtered and a card has local state?

Vue uses the `key` to match existing components to new ones after a re-render. When you filter the list, the index of each event shifts — for example, event index 2 becomes index 0. Vue sees that index 0 still exists and *reuses* the component that was already mounted there, carrying over any local state it held (such as an open details panel). This means a card's local state ends up attached to the wrong event. Using the stable, unique `e.id` as the key prevents this: Vue unmounts cards that disappeared and mounts new ones at the right position.

---

## 2. Why is `visible` a computed and not a function called from the template?

A computed is **cached** based on its reactive dependencies (`search` and `category`). It only re-runs when one of those values actually changes, so every keystroke in the search box triggers exactly one re-evaluation — no matter how many other things cause the component to re-render. A plain function in the template has no cache: it would be called on *every* render, even when unrelated state changes. With the computed, Vue skips the filtering work entirely if neither `search` nor `category` has changed.

---

## 3. `EventCard` emits `book` instead of changing `event.booked` directly. Give two reasons.

**First**, props are read-only in Vue — mutating a prop directly (`props.event.booked++`) breaks the one-way data flow and Vue will warn you. A child component should never own state it received from a parent.

**Second**, keeping the mutation in the owner (`App.vue`) makes `EventCard` reusable and independently testable. The card only needs to *report intent* ("the user wants to book this event"); the parent decides whether to allow it, how to update the data, and what side effects to run (e.g. saving to `localStorage`).

---

## 4. You write `const { booked } = reactive(event); booked++`. The page does not change. Why?

Destructuring a **primitive** (a `number`) out of a reactive object copies the current value into a plain local variable — it loses its reactive link to the object. Incrementing `booked` changes only that local variable; the `booked` property on the reactive object is never touched. Because Vue's reactivity system tracks accesses and mutations on the *object's own properties*, it never detects the change and does not schedule a re-render.

---

## 5. Where does `myBookings` live, and why not in `EventCard` or `EventList`?

`myBookings` lives in **`App.vue`**, which is the lowest common ancestor of all components that need it. `App` owns the `events` array and is therefore the only component that should mutate it (via `onBook`). The booking count is also displayed in the `<header>` inside `App`, so the data must be at that level to be accessible to both the header and the list. Placing `myBookings` in `EventCard` would scope it to a single card (lost on unmount), and in `EventList` it would be hidden from the header — both violate the rule that state belongs in the component that owns everything depending on it.
