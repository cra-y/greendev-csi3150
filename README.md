# Greenhouse Sensor Logger: UI Architecture (Assignment 2A)

This app is a client-side telemetry logger for greenhouse bays. This is CSI 3150 Assignment 2A.
The implementation, assignment 2B, will be HTML + CSS + JS with localStorage.

### Assumptions
* Fahrenheit only.
* Bounds and steps for each metric:
* pH 0–14 / 0.1
* EC 0–6 / 0.01
* Temp 0–120 / 0.1
* RH 0–100 / 0.1
* Bay names: A-D
* Choosing OR instead of AND: The spec uses AND, which means one dangerous values would show a warning (i.e a value such as pH 2.0 would only be a warning under AND). OR was also used in class, so this will reflect our coursework.

---

## Section 4.1: The ≤4-Click User Journey Funnel

- **Starting State:**
Saved readings load from localStorage on DOMContentLoaded. If there are none, #empty-state shows 0
- **Action 1: Data Entry & Real-Time Constraint Check:** 
User chooses a bay and types the required four values. input event toggles .field-warning or .field-danger. You can still submit the inputs with the warnings
- **Action 2: Log Submission:** 
sumbit > preventDefault() > JS checks the bounds. #form-error shows if it fails and it does not save. If it does not fail, parseFloat > object is built > object is classified > shows in the history and form resets
- **Action 3: Output Telemetry Inspection:**
A new list appears in #historical with badge text and the color representing its status. Log count is also updated and the empty state is hidden
- **Action 4: Output Resolution:**
Each card has an individual delete button and also has a "purge all" button to delete all historical entries
- **Terminal State:**
Finished readings are removed, and everything else stays on localStorage. The form is reset as well. Empty state returns if everytihng is purged

---

## Section 4.2: Don Norman Usability & Constraint Audit

| Norman Principle | System Context | Implementation Mechanism (DOM / HTML / JS) |
|---|---|---|
| Gulf of Execution | Typing numeric sensor metrics (pH, EC, Temp, RH) | Units will be labeled, placeholder examples, steps to make precise inputs. Bay has dropdowns to ensure no errors are made |
| Gulf of Evaluation | Ingesting values after clicking "Record Reading" | Status badge: Appears around card; yellow on warning, red on critical. Normal shows normal outline. |
| System Constraint | Water pH boundaries (0.0 ≤ pH ≤ 14.0) | type="number", min="0", max="14", step="0.1", required. The submit handler checks each value against min/max in JS, shows #form-error and does not save if the check fails |
| Visual Feedback State | Real-time input error / out-of-bounds | the input event checks the value against the thresholds and shows .field-warning or .field-danger if necessary. |
| Error Recovery | Accidental or test data submissions | Purge button incorporated to delete all entries. Singular delete button under individual entries. Confirmation included before purge button to prevent accidental deletion.  |

---

## Section 4.3: Component, DOM Tree & Event Target Architecture

**Legend:** `[STATIC]` = in index.html at load · `[DYNAMIC]` = created/changed by JS ·
`(Event: type)` = listener attached here · `(CSS: …)` = major layout property only

```
index.html
├── <header class="site-header"> (CSS: flexbox; row, space-between, align-center) [STATIC]
│   ├── <h1> (Text: GreenDev) [STATIC]
│   └── <p> (Text: "Active logs: ") + <span id="log-count"> [STATIC — number updated by JS]
│
├── <main> (CSS: grid; 2 columns / 1 column under 768px (mobile)) [STATIC]
│   │
│   ├── <section id="entry-panel"> (CSS: flexbox; column, gap 1rem) [STATIC]
│   │   ├── <h2> (Text: Record Sensor Reading) [STATIC]
│   │   ├── <div id="form-error" class="alert hidden" role="alert"> [STATIC — shown/hidden by toggling .hidden]
│   │   └── <form id="input-form" novalidate> (Event: submit) [STATIC]
│   │       ├── <label for="bay"> (Text: "Greenhouse Bay") + <select id="bay" required> (no event) [STATIC]
│   │       │   ├── <option value="" disabled selected> (Text: "Select a bay")
│   │       │   └── <option value="A"> … <option value="D">
│   │       ├── <label for="ph"> (text: "pH 0-14") + <input type="number" id="ph" min="0" max="14" step="0.1" placeholder="6.1" required> (Event: input > toggles .field-warning/.field-danger) [STATIC]
│   │       ├── <label for="ec"> (text: "EC (mS/cm)") + <input type="number" id="ec" min="0" max="6" step="0.01" placeholder="2.02" required> (Event: input > toggles .field-warning/.field-danger) [STATIC]
│   │       ├── <label for="temp"> (text: "Temp F") + <input type="number" id="temp" min="0" max="120" step="0.1" placeholder="66.2" required> (Event: input > toggles .field-warning/.field-danger) [STATIC]
│   │       ├── <label for="rh"> (text: "RH 0-100%") + <input type="number" id="rh" min="0" max="100" step="0.1" placeholder="81" required> (Event: input > toggles .field-warning/.field-danger) [STATIC]
│   │       └── <button type="submit"> (Text: "Record Reading") [STATIC]
│   │
│   └── <section id="log-history"> (CSS: flexbox; column, gap 1rem) [STATIC]
│       ├── <header> (CSS: flexbox; row, space-between) [STATIC]
│       │   ├── <h2> (Text: Data Log History) [STATIC]
│       │   └── <button id="purge-all"> (Text: purge all) (Event: click) [STATIC]
│       ├── <p id="empty-state"> (Text: "No entries logged yet") [STATIC — hidden with .hidden when logs exist]
│       └── <ul id="historical" aria-live="polite"> (CSS: flexbox; column, gap) (Event: click) [STATIC]
│           └── <li class="log-entry status-optimal | status-warning |  status-critical" data-id="..."> (CSS: flexbox; column, gap) [DYNAMIC]
│               ├── <h3> (Text: "Bay ___ · <time>") [DYNAMIC]
│               ├── <dl class="readings"> (CSS: grid; 2 columns) [DYNAMIC] pH, EC, Temp, RH 
│               ├── <span class="status-badge optimal | warning | status-critical"> (Text: "optimal | warning | critical hazard") [DYNAMIC]
│               └── <button class="btn-delete" data-id="..."> (Text: "Delete") [DYNAMIC]
│
└── <footer> (CSS: Flexbox: row, centered) [STATIC]
   └── <p> (Text: "Copyright 2026 GreenDev CSI 3150") [STATIC]
```
On DOMContentLoaded, load readings from localStorage with JSON.parse and render them

---

## Section 4.4: Data Schema

<!-- TODO: fill in keys, types, and a comment for each.
     Remember: input.value is always a string. Decide what type you STORE. -->

```js
const sensorReadingSchema = {
  id: "String", // Date.now().toString()
  bay:  "String", //A-D
  timestamp: "String", //new Date().toISOString()
  ph: "Number",
  ec: "Number",
  temp: "Number", //F
  humidity: "Number",
  status: "String" //optimal, warning, critical hazard
};

//Stored using the localStorage key and JSON.stringify or JSON.parse
```
