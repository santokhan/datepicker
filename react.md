## ReactJS Calendar Component

```jsx
import React, { useState, useMemo } from "react";
import "./Calendar.css"; // Use the same CSS converted from your Vue scoped style

const daysOfWeek = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
const monthNames = [
  "January", "February", "March", "April", "May", "June",
  "July", "August", "September", "October", "November", "December"
];
const offWeekDays = [0, 6]; // Sunday and Saturday

const Calendar = ({ value, onChange }) => {
  const today = new Date();
  const [month, setMonth] = useState(today.getMonth());
  const [year, setYear] = useState(today.getFullYear());
  const [selectedDate, setSelectedDate] = useState(value || null);

  // Generate month days
  function* monthDays(year, month) {
    const firstDay = new Date(year, month, 1).getDay();
    const daysInMonth = new Date(year, month + 1, 0).getDate();

    for (let i = 0; i < firstDay; i++) yield null;
    for (let day = 1; day <= daysInMonth; day++) yield day;
    const total = firstDay + daysInMonth;
    const remaining = (7 - (total % 7)) % 7;
    for (let i = 0; i < remaining; i++) yield null;
  }

  function generateCalendar(year, month) {
    const weeks = [];
    let week = [];
    for (const day of monthDays(year, month)) {
      week.push(day);
      if (week.length === 7) {
        weeks.push(week);
        week = [];
      }
    }
    return weeks;
  }

  const weeks = useMemo(() => generateCalendar(year, month), [year, month]);

  // Navigation
  const prevMonth = () => {
    if (month === 0) {
      setMonth(11);
      setYear(year - 1);
    } else setMonth(month - 1);
  };

  const nextMonth = () => {
    if (month === 11) {
      setMonth(0);
      setYear(year + 1);
    } else setMonth(month + 1);
  };

  // Selection and validation
  const selectDate = (day) => {
    if (!day) return;
    const date = new Date(year, month, day);
    setSelectedDate(date);
    onChange?.(date);
    console.log("Selected:", date.toISOString().split("T")[0]);
  };

  const isSelected = (day) => {
    if (!selectedDate || !day) return false;
    return (
      selectedDate.getFullYear() === year &&
      selectedDate.getMonth() === month &&
      selectedDate.getDate() === day
    );
  };

  const isToday = (day) => {
    if (!day) return false;
    return (
      day === today.getDate() &&
      month === today.getMonth() &&
      year === today.getFullYear()
    );
  };

  const isDisabled = (day) => {
    if (!day) return true;
    const date = new Date(year, month, day);
    if (date < today && !isToday(day)) return true;
    if (offWeekDays.includes(date.getDay())) return true;
    return false;
  };

  return (
    <div className="calendar">
      <div className="header">
        <select value={month} onChange={(e) => setMonth(Number(e.target.value))}>
          {monthNames.map((m, i) => (
            <option key={i} value={i}>{m}</option>
          ))}
        </select>
        <input
          type="number"
          value={year}
          min={1900}
          max={2100}
          onChange={(e) => setYear(Number(e.target.value))}
        />
        <button onClick={prevMonth}>◀</button>
        <button onClick={nextMonth}>▶</button>
      </div>

      <table>
        <thead>
          <tr>{daysOfWeek.map((d) => <th key={d}>{d}</th>)}</tr>
        </thead>
        <tbody>
          {weeks.map((week, i) => (
            <tr key={i}>
              {week.map((day, j) => (
                <td
                  key={j}
                  className={`${!day ? "empty" : ""} ${isDisabled(day) ? "disabled" : ""}`}
                  onClick={() => !isDisabled(day) && selectDate(day)}
                >
                  {day && (
                    <button
                      type="button"
                      className={`${isSelected(day) ? "selected" : ""} ${isToday(day) ? "today" : ""}`}
                    >
                      {day}
                    </button>
                  )}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default Calendar;
```

---

## Calendar CSS (Calendar.css)

```css
.calendar {
  width: 320px;
  padding: 10px;
  font-size: 0.75rem;
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 1px 2px rgba(44, 39, 56, 0.03);
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.calendar table {
  width: 100%;
  border-collapse: collapse;
  text-align: center;
}

.calendar td,
.calendar th {
  padding: 6px;
}

.calendar td.empty {
  background: white;
}

.calendar td.disabled button {
  color: #aaa;
  pointer-events: none;
}

.calendar td button {
  width: 30px;
  height: 30px;
  border: none;
  background: transparent;
  cursor: pointer;
  border-radius: 50%;
}

.calendar td button:hover {
  background: #f0f0f0;
}

.calendar td button.selected {
  background: #4caf50;
  color: #073e08;
  font-weight: 500;
}

.calendar td button.today {
  border: 1px solid #4caf50;
  font-weight: bold;
}
```

---

## 📄 Markdown Documentation

### Calendar Component

A fully functional **ReactJS calendar** component with:

* Month/year navigation
* Day selection
* Highlighting today
* Disabled past dates
* Recurring off-days (weekends)

---

### Props

| Prop       | Type     | Description                          |
| ---------- | -------- | ------------------------------------ |
| `value`    | Date     | Optional: initial selected date      |
| `onChange` | function | Callback triggered on date selection |

---

### Usage

```jsx
import React, { useState } from "react";
import Calendar from "./Calendar";

export default function App() {
  const [selectedDate, setSelectedDate] = useState(null);

  return (
    <div>
      <h1>Selected Date: {selectedDate?.toDateString() || "None"}</h1>
      <Calendar value={selectedDate} onChange={setSelectedDate} />
    </div>
  );
}
```

---

### Features

1. Navigate months via arrows or dropdown.
2. Select a date with visual feedback.
3. Today's date is highlighted.
4. Past dates (except today) are disabled.
5. Weekends are disabled as recurring off-days.
