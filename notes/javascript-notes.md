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
// BROKEN — missing closing " on the class name
`<span class="pagination-page>Page ${n}</span>`// FIXED
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

## Resources

- [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [MDN – Array.prototype.slice()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- [MDN – Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- [MDN – Math.max()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max)
