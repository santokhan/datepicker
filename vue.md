## 1. **Template Section**

```vue
<template>
  <div class="calendar">
    <div class="header">
      <select v-model.number="month">...</select>
      <input v-model.number="year" type="number" min="1900" max="2100">
      <button @click="prevMonth">◀</button>
      <button @click="nextMonth">▶</button>
    </div>

    <table>
      <thead>
        <tr>
          <th v-for="d in daysOfWeek" :key="d">{{ d }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(week, i) in weeks" :key="i">
          <td v-for="(day, j) in week" :key="j" :class="{ empty: !day, disabled: isDisabled(day) }"
            @click="!isDisabled(day) && selectDate(day)">
            <button v-if="day" type="button" :class="{ selected: isSelected(day), today: isToday(day) }">
              {{ day }}
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
```

**Explanation:**

* **Header Section**

  * `select` → lets the user choose the month
  * `input` → lets the user enter the year
  * `<button>` → navigation for previous/next month

* **Table Section**

  * `<thead>` → displays day names (Sun, Mon, …)
  * `<tbody>` → loops through weeks and days
  * `td.empty` → empty cells for alignment
  * `td.disabled` → disabled dates (off days or past dates)
  * `<button>` → clickable date, highlighted if selected or today

---

## 2. **Script Section**

```js
import { computed, defineEmits, ref } from "vue"
const emit = defineEmits(["update:modelValue"])
const today = new Date()
const month = ref(today.getMonth())
const year = ref(today.getFullYear())
const selectedDate = ref(null)
const daysOfWeek = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"]
const monthNames = ["January", ..., "December"]
const offWeekDays = [0, 6] // Sunday=0, Saturday=6
```

**Explanation:**

* `ref` → reactive variables for month, year, and selected date
* `defineEmits` → to emit selected date to parent component
* `today` → current date reference
* `offWeekDays` → recurring disabled days (weekends)

---

### **Generator function: `monthDays`**

```js
function* monthDays(year, month) {
  const firstDay = new Date(year, month, 1).getDay()
  const daysInMonth = new Date(year, month + 1, 0).getDate()

  for (let i = 0; i < firstDay; i++) yield null
  for (let day = 1; day <= daysInMonth; day++) yield day
  const total = firstDay + daysInMonth
  const remaining = (7 - (total % 7)) % 7
  for (let i = 0; i < remaining; i++) yield null
}
```

**Explanation:**

* Generates all days of the month **including empty padding cells**
* `firstDay` → index of the first day of the month (0–6)
* `daysInMonth` → number of days in the month
* `yield null` → empty cells before and after actual days for proper table alignment

---

### **Generate weeks: `generateCalendar`**

```js
function generateCalendar(year, month) {
  const weeks = []
  let week = []
  for (const day of monthDays(year, month)) {
    week.push(day)
    if (week.length === 7) {
      weeks.push(week)
      week = []
    }
  }
  return weeks
}
const weeks = computed(() => generateCalendar(year.value, month.value))
```

**Explanation:**

* Converts the flat list of days into **weeks** (arrays of 7 elements)
* `weeks` → reactive computed property used in the template

---

### **Navigation Functions**

```js
function prevMonth() {
  if (month.value === 0) { month.value = 11; year.value -= 1 }
  else month.value -= 1
}

function nextMonth() {
  if (month.value === 11) { month.value = 0; year.value += 1 }
  else month.value += 1
}
```

**Explanation:**

* Changes month and adjusts year if needed

---

### **Date Selection & Checks**

```js
function selectDate(day) {
  selectedDate.value = new Date(year.value, month.value, day)
  emit("update:modelValue", selectedDate.value)
}

function isSelected(day) { ... }
function isToday(day) { ... }
function isDisabled(day) { ... }
```

**Explanation:**

* `selectDate(day)` → sets selected date and emits it
* `isSelected(day)` → highlights selected date
* `isToday(day)` → highlights current date
* `isDisabled(day)` → disables past dates and weekends

---

## 3. **Style Section**

```css
.calendar { ... }
.header { ... }
td.disabled button { color: #aaa; pointer-events: none; }
td button.selected { background: #4caf50; color: #073e08; }
td button.today { border: 1px solid #4caf50; }
```

**Explanation:**

* Styles for calendar container, header, table, buttons
* `.disabled` → greyed out and unclickable
* `.selected` → green background for selected date
* `.today` → outlined with green border

---

### ✅ **Summary**

This component is:

* **Reactive:** updates calendar on month/year changes
* **Selectable:** allows selecting a date and emits it
* **User-friendly:** disables past and off-days, highlights today
* **Flexible:** you can change off-days, styling, or date validation
