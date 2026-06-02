# JavaScript Notes

> Key concepts and things I've learned about JavaScript during placement

---

## Variables

```js
const  // can't be reassigned – use by default
let    // can be reassigned – use when value will change
var    // old way – avoid in modern JS
```

---

## Functions

```js
// Regular function
function greet(name) {
  return `Hello, ${name}!`;
}

// Arrow function
const greet = (name) => `Hello, ${name}!`;
```

---

## Array Methods (useful ones)

```js
.map()     // transform every item, returns new array
.filter()  // keep only items that pass a test
.find()    // return the first item that matches
.forEach() // loop through items (no return)
.reduce()  // boil an array down to a single value
.slice()   // return a portion of an array without changing the original
```

---

## Async / Await

```js
const fetchData = async () => {
  try {
    const response = await fetch("https://api.example.com/data");
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error("Error:", error);
  }
};
```

---

## Notes from Placement

### Pagination — slicing data across pages

Pagination means splitting a large array into smaller "pages" and only showing one page at a time. The key is to slice the data _after_ filtering it, so your page counts are always accurate.

```js
const PER_PAGE = 50; // how many items per page

function paginate(items, tabKey) {
  const currentPage = paginationState[tabKey]; // which page are we on?
  const totalPages = Math.ceil(items.length / PER_PAGE);

  const start = currentPage * PER_PAGE;
  const end = start + PER_PAGE;
  const pageItems = items.slice(start, end); // only the items for this page

  const from = start + 1;
  const to = Math.min(end, items.length);
  const total = items.length;

  // Build the Next/Back controls HTML
  const controlsHtml =
    totalPages <= 1
      ? ""
      : `
    <div class="pagination-controls">
      <span class="pagination-info">Showing ${from}–${to} of ${total}</span>
      <div class="pagination-buttons">
        <button
          class="pagination-btn"
          onclick="goToPage('${tabKey}', ${currentPage - 1})"
          ${currentPage === 0 ? "disabled" : ""}>
          ← Back
        </button>
        <span class="pagination-page">Page ${
          currentPage + 1
        } of ${totalPages}</span>
        <button
          class="pagination-btn"
          onclick="goToPage('${tabKey}', ${currentPage + 1})"
          ${currentPage === totalPages - 1 ? "disabled" : ""}>
          Next →
        </button>
      </div>
    </div>`;

  return { pageItems, controlsHtml };
}
```

**Important:** Always filter your data _before_ passing it to `paginate()`. If you paginate unfiltered data and skip rows during rendering, the page counts will be wrong.

---

### Clamping numbers with Math.max / Math.min

Clamping means keeping a number within a valid range. It's a standard technique used in pagination, scroll handlers, game engines, and carousels.

```js
// Prevent page number going below 0
paginationState[tabKey] = Math.max(0, newPage);

// Prevent a value going above a maximum
const clamped = Math.min(value, maxValue);

// Clamp between a min and max
const clamped = Math.max(min, Math.min(value, max));
```

When used in a `goToPage()` function, `Math.max(0, newPage)` means the page can never go below 0, no matter what value gets passed in.

---

### Grouping array items with reduce()

`reduce()` can build an object that groups items by a shared property — useful for organising entities by type.

```js
// Groups an array of entities by their 'type' property
// Result: { IfcWall: [...], IfcDoor: [...], IfcWindow: [...] }
const byType = entities.reduce((acc, entity) => {
  if (!acc[entity.type]) acc[entity.type] = [];
  acc[entity.type].push(entity);
  return acc;
}, {});

// Then loop through the groups
for (const [type, items] of Object.entries(byType)) {
  // render a collapsible group for each type
}
```

---

### Template literal bugs — unclosed quotes

Inside template literals, a missing closing quote on a CSS class attribute causes everything after it to be misread as part of the string. The result looks like a render failure but the real cause is one character.

```js
// BROKEN, missing closing " on the class name
`<span class="pagination-page>Page ${n}</span>` // FIXED
`<span class="pagination-page">Page ${n}</span>`;
```

Always use double quotes inside template literals for HTML attributes, it keeps things consistent and avoids this type of bug.

---

### Resetting state when new data loads

When a user loads a new file, any saved state (like which page they were on) should be reset otherwise they might land on a page that doesn't exist in the new data.

```js
// Reset all pagination to page 0 when a new file is loaded
Object.keys(paginationState).forEach((key) => (paginationState[key] = 0));
```

---

### Error handling — guard clauses

A guard clause is an early return that stops a function if required data is missing. It protects the lines below it from crashing.

**Guard clause order — always the same pattern:**

1. Define the container
2. Guard the container
3. Guard `window.state`
4. Guard the specific data the function needs
5. Guard for empty results last

```js
function displayPropertiesTab() {
  const container = document.getElementById("propertiesContent");

  if (!container) {
    console.error("Properties container not found");
    return;
  }

  if (!window.state) {
    console.error("State not ready");
    return;
  }

  if (!window.state.allEntities) {
    container.innerHTML = `<p class="error-banner">No entity data found. Please reload the file.</p>`;
    return;
  }

  if (window.state.allEntities.length === 0) {
    container.innerHTML = `<p class="error-banner">No entities found in this model.</p>`;
    return;
  }

  // safe to proceed
}
```

**Two audiences for error messages:**

- Users → `container.innerHTML` with a plain English message
- Developers → `console.error()` with a specific technical message

---

### DRY — Don't Repeat Yourself

Instead of writing six nearly identical lines, use a dynamic value to target the right element:

```js
// BEFORE — repeated six times
document.getElementById("entitiesContent").innerHTML = "...";
document.getElementById("propertiesContent").innerHTML = "...";
// ... four more times

// AFTER — one dynamic line
const container = document.getElementById(tabName + "Content");
container.innerHTML = "...";
```

---

### Checking if an object is empty

An empty object `{}` is still truthy in JavaScript — `!obj` won't catch it.

```js
// WRONG: {} is truthy, this won't catch an empty object
if (!window.state.modelInfo.project) { ... }

// CORRECT:  checks whether the object has any keys
if (Object.keys(window.state.modelInfo.project).length === 0) { ... }

// FULL GUARD, check both: does it exist AND is it non-empty?
if (!window.state.modelInfo.project ||
    Object.keys(window.state.modelInfo.project).length === 0) { ... }
```

---

### Variables vs strings — a common mistake

Inside a function, using quotes around a parameter name means you're searching for a _literal_ element ID, not using the variable passed in.

```js
function showSkeleton(containerId) {
  // WRONG: searches for an element literally called "containerID"
  const container = document.getElementById("containerID");

  // CORRECT: uses whatever was passed into containerId
  const container = document.getElementById(containerId);
}
```

---

### Navigation history stack back button pattern

An array used as a stack lets users navigate forward and back through views. `push` saves the current state before moving forward. `pop` restores it when going back. This naturally supports multiple drill-down levels.

```js
// Declare at module scope so it persists across function calls
const entityNavStack = [];

// Before navigating forward, save the current HTML
entityNavStack.push(document.getElementById("entitiesContent").innerHTML);

// When going back, restore the saved HTML
window.entityNavBack = function () {
  if (!entityNavStack.length) return;
  document.getElementById("entitiesContent").innerHTML = entityNavStack.pop();
};
```

Important: this variable must be declared at the top level of the file, outside any function. If it is declared inside a function it resets to an empty array every time that function runs.

---

### setTimeout(fn, 0) yielding to the render thread

`setTimeout` with a delay of 0 does not run immediately. It pushes the callback to run after the browser has finished its current work, including painting the screen. This is useful when you want the user to see a "Loading..." message before heavy JavaScript runs.

```js
panel.innerHTML = `<p>Loading...</p>`;

// Without setTimeout, the browser tries to paint AND run the data build at the same time
// The "Loading..." message never appears
setTimeout(async function () {
  await window.lazyLoadEntities(window.state?.currentModelID);
  panel.innerHTML = buildViewerEntitiesHTML();
}, 0);
```

This technique is called "yielding to the render thread" and is a standard pattern in Vanilla JavaScript.

---

### Module-scope vs function-scope variables

Variables declared inside a function are reset every time that function runs. Variables declared at module scope (outside all functions, at the top of the file) persist for the lifetime of the page.

```js
// WRONG: resets to [] every time the function is called
function showEntityDetail(entity) {
  const entityNavStack = []; // starts fresh every call
  entityNavStack.push(...);
}

// CORRECT: persists across all function calls
const entityNavStack = []; // declared once at the top of the file

function showEntityDetail(entity) {
  entityNavStack.push(...); // uses the same array every time
}
```

---

### Guard clauses must check the right piece of state

Using the wrong variable as a guard means your check passes or fails at the wrong time.

```js
// WRONG: currentModelID is set by the 3D loader, not the IFC parser
// The file may be loading but the parsed data is not ready yet
if (!window.state?.currentModelID) {
  panel.innerHTML = "Load an IFC file first.";
  return;
}

// CORRECT: allEntities.length confirms the parsed data is actually ready
if (!window.state?.allEntities?.length) {
  panel.innerHTML = "Load an IFC file first.";
  return;
}
```

---

## Resources

- [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [MDN Array.prototype.slice()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- [MDN Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- [MDN Math.max()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max)
- [MDN Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)
- [MDN setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout)
