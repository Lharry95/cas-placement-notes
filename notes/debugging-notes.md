# Debugging Notes

> Key concepts and bugs I've encountered and solved during placement

---

## Week 3

### Duplicate Buttons Rendering on Page

**What happened:** The Footprints tab was showing double buttons: All Doors, Inward Only, Outward Only, and Unknown Direction were all appearing twice.

**Root cause:** The button code existed in both the HTML file and in `UI.JS`, causing them to render twice.

**Fix:** Removed the button code from the HTML and kept it only in `UI.JS`.

```js
// Keep button logic in UI.JS only, not in the HTML
// Defining buttons in both places causes them to render twice
```

**What I learned:** When UI elements are defined in both HTML and JavaScript, they can render twice. Logic like this belongs in one place. If it's being generated dynamically, keep it in JavaScript only.

---

### Collapsible Panels Starting Open Instead of Collapsed

**What happened:** Property set panels were loading in an open/expanded state by default, when they should start collapsed and only open when clicked.

**Root cause:** No default collapsed state was set on the HTML elements. The CSS classes controlling visibility weren't applied on load.

**Fix:** Added `collapsed` to the property set div class name and `hidden` to the collapsible content div class name directly in the HTML.

```html
<!-- Before -->
<div class="property-set">
  <div class="collapsible-content">
    <!-- content -->
  </div>
</div>

<!-- After -->
<div class="property-set collapsed">
  <div class="collapsible-content hidden">
    <!-- content -->
  </div>
</div>
```

**What I learned:** Default UI state can be set through CSS class names directly in the HTML. JavaScript only needs to toggle the classes from there. You don't need JS to control what something looks like on first load.

---

### App Freezes When Large IFC File Is Loaded

**What happened:** When a large IFC file is loaded, only the 3D viewer displays correctly. Clicking any other tab (e.g. Entities) causes the application to freeze.

**Root cause:** Under investigation. It is likely a performance issue with how tab content is loaded alongside a large file.

**Fix:** Escalated to the team. Not yet resolved.

**What I learned:** Large file handling may require performance optimisation such as lazy loading tab content rather than loading everything at once.

---

### No Back Button in Entities View

**What happened:** When navigating into a specific entity in the Entities view, there is no way to return to the entities list. No back button or navigation exists.

**Root cause:** Navigation state inside the panel isn't being tracked, so there's no history to go back to.

**Fix:** Resolved in Week 6 using a navigation history stack.

**What I learned:** Panel-based navigation needs to track state so users can move backwards through views. This is different from browser navigation and needs to be handled manually in JavaScript.

---

## Week 4

### Pagination Next Button Stopped Working

**What happened:** The Next button in the pagination controls stopped working entirely after it was added.

**Root cause:** A missing closing `"` on the `pagination-page` CSS class name inside a template literal. One character caused the entire button's HTML to be malformed.

**Fix:** Added the missing closing quote in the template literal.

```js
// BROKEN: missing closing " on the class name
`<span class="pagination-page>Page ${n}</span>` // FIXED
`<span class="pagination-page">Page ${n}</span>`;
```

**What I learned:** A single missing character inside a template literal can cause HTML to be completely misread. When a rendered element isn't behaving, inspect the actual HTML output in DevTools. The cause is often much smaller than the symptom.

---

### Page Numbers Going Out of Bounds on Smaller Files

**What happened:** When switching to a smaller IFC file, the app tried to load a page number that didn't exist in the new file, causing an error.

**Root cause:** `goToPage()` wasn't checking that the new page number was within a valid range. It could go below 0 if a user had been on a later page with a larger file.

**Fix:** Used `Math.max(0, newPage)` to clamp the page number so it can never go below 0.

```js
paginationState[tabKey] = Math.max(0, newPage);
```

**What I learned:** Numbers that drive UI state (like page numbers) should always be clamped to a valid range. `Math.max` and `Math.min` are the standard tools for this.

---

### `groupId is not defined` Breaking the Entities Tab

**What happened:** The entire Entities tab stopped rendering and threw a `groupId is not defined` error.

**Root cause:** A single missing variable declaration `const groupId` inside `displayEntitiesTab()`. Every line referencing `groupId` after it was breaking.

**Fix:** Added the missing `const groupId` declaration in the right place inside the loop.

**What I learned:** A missing variable declaration breaks every line that references it. When a function fails completely, check whether all variables used inside it are actually declared.

---

## Week 5

### Catch Block Referencing `container` Before It Was Declared

**What happened:** The `catch` block in `lazyLoadTabData` was trying to use a variable called `container` that was declared outside the catch. If the error happened before that line ran, `container` didn't exist yet.

**Root cause:** `container` was referenced in the catch block before it had been assigned.

**Fix:** Moved the `container` declaration inside the catch block so it's always available when needed.

```js
// BEFORE: container might not exist yet
} catch (error) {
  if (container) {
    container.innerHTML = `<p class="error-banner">Error loading data.</p>`;
  }
}

// AFTER: declare it fresh inside the catch
} catch (error) {
  const container = document.getElementById(tabName + "Content");
  if (container) {
    container.innerHTML = `<p class="error-banner">Error loading data.</p>`;
  }
  console.error("Error loading tab data:", error);
}
```

**What I learned:** Variables declared outside a `try` block aren't guaranteed to exist inside the `catch`. Always declare what you need inside the block that uses it.

---

### Incorrect Empty Check on an Object

**What happened:** A guard clause using `!window.state.modelInfo.project` wasn't catching cases where the project object existed but was empty (`{}`).

**Root cause:** An empty object `{}` is truthy in JavaScript. `!obj` only catches `null` or `undefined`, not an object with no keys.

**Fix:** Used `Object.keys().length === 0` to correctly check whether the object has any content.

```js
// WRONG: {} is truthy, this won't catch an empty object
if (!window.state.modelInfo.project) { ... }

// CORRECT
if (!window.state.modelInfo.project ||
    Object.keys(window.state.modelInfo.project).length === 0) { ... }
```

**What I learned:** In JavaScript, an empty object is not falsy. Always use `Object.keys(obj).length === 0` when you need to check if an object has no data.

---

### String Passed Instead of Variable Inside `showSkeleton()`

**What happened:** `showSkeleton()` wasn't finding the correct container element. It was searching for a literal element with the ID `"containerID"` instead of using the value passed in.

**Root cause:** The parameter name was accidentally written in quotes inside `document.getElementById()`, turning the variable into a hardcoded string.

**Fix:** Removed the quotes so the variable is used instead of a literal string.

```js
// WRONG: searches for an element literally called "containerID"
const container = document.getElementById("containerID");

// CORRECT: uses whatever was passed into the function
const container = document.getElementById(containerId);
```

**What I learned:** Writing a variable name in quotes makes it a string. JavaScript will search for an element with that exact text as its ID. Always double-check that parameter names inside functions aren't accidentally quoted.

---

## Week 6

### 3D Canvas Blank After HTML Restructure

**What happened:** After restructuring the HTML, the 3D viewer canvas went completely blank.

**Root cause:** Two divs both had `id="viewer-container"`. `getElementById` only finds the first one, so Three.js was targeting the wrong element. No error was thrown.

**Fix:** Removed the duplicate wrapper div so only one element has `id="viewer-container"`.

**What I learned:** Duplicate IDs cause silent failures. `getElementById` finds the first match and ignores any others. Always keep IDs unique. When something renders blank with no console error, check for duplicate IDs in the HTML.

---

### Viewer Collapse Toggle Not Working

**What happened:** Clicking the viewer collapse button had no visible effect.

**Root cause:** The code was transitioning `height`, which does not work reliably on elements inside a flexbox layout.

**Fix:** Switched to a `max-height` transition instead.

```css
/* Set a large max-height in CSS */
#viewer-body {
  max-height: 2000px;
  transition: max-height 0.3s ease;
  overflow: hidden;
}
```

```js
// Collapse
viewerBody.style.maxHeight = "0px";

// Expand
viewerBody.style.maxHeight = "2000px";
```

**What I learned:** `height` transitions do not animate reliably on flex children. `max-height` works because the browser can calculate the transition between two concrete values. `height: auto` to `0` does not work because `auto` has no concrete value to animate from.

---

### Bottom Tabs Showing "Load IFC First" Despite File Being Loaded

**What happened:** After loading an IFC file and clicking a bottom viewer tab, the panel showed "Load an IFC file first" even though a file was already loaded.

**Root cause:** The guard clause was checking `currentModelID`, which is set by the 3D loader. The IFC parser sets `allEntities`, and that was not ready yet when the guard ran.

**Fix:** Changed the guard to check `allEntities.length` instead.

```js
// WRONG: checks the 3D loader state, not the parsed data state
if (!window.state?.currentModelID) { ... }

// CORRECT: checks whether the IFC parser has actually finished
if (!window.state?.allEntities?.length) { ... }
```

**What I learned:** Guard clauses must check the right piece of state. Using the wrong variable means the check passes or fails at the wrong time.

---

### Bottom Tabs Required Clicking Another Tab First

**What happened:** On first click, a bottom viewer tab showed no data. Clicking a different tab and then back made it work.

**Root cause:** The guard clause ran before `lazyLoadEntities` had finished populating state. The check happened too early.

**Fix:** Moved the guard inside the `setTimeout` callback, after the `await lazyLoadEntities` call.

```js
setTimeout(async function () {
  await window.lazyLoadEntities(window.state?.currentModelID);

  // Guard runs here, AFTER data has loaded
  if (!window.state?.allEntities?.length) {
    panel.innerHTML = "Load an IFC file first.";
    return;
  }

  panel.innerHTML = buildViewerEntitiesHTML();
}, 0);
```

**What I learned:** When async data is involved, guards must run after the `await`, not before it. Putting the guard before the `await` means it checks state that has not been populated yet.

---

### `ReferenceError: isSidebarOpen` After Editing the File

**What happened:** The sidebar toggle threw a `ReferenceError: isSidebarOpen is not defined`.

**Root cause:** The `let isSidebarOpen` declaration had been accidentally deleted from the file during editing.

**Fix:** Re-added `let isSidebarOpen = true` at module scope at the top of `ui.js`.

**What I learned:** State variables that are referenced across multiple functions must be declared at module scope. If they are missing, every function that references them throws a ReferenceError.

---

### Main Content Not Expanding When Sidebar Collapsed

**What happened:** Collapsing the sidebar made it visually smaller but the main content area did not expand to fill the freed space.

**Root cause:** Changing `width` on the sidebar element does not affect the grid column it sits in. The grid layout controls how space is distributed, not the element's own width.

**Fix:** Updated `grid-template-columns` on the parent layout element directly via JavaScript on every toggle.

```js
// WRONG: only shrinks the sidebar, does not affect the grid column
sidebar.style.width = "48px";

// CORRECT: updates the grid so the main content column expands automatically
document.querySelector(".app-layout").style.gridTemplateColumns = "48px 1fr";
```

**What I learned:** In a CSS Grid layout, the grid column controls space distribution. Changing an element's `width` inside a grid column has no effect on the surrounding layout. Always update `gridTemplateColumns` on the parent to change how space is shared.

---

### Post-Merge Breakage After Pulling Teammate Code

**What happened:** After merging a teammate's code, three things broke at once: a syntax error on load, all tabs hidden immediately on page load, and the bottom viewer tabs reverting to broken behaviour.

**Root cause:**

1. `entityNavStack` was declared before the `import` statement. In JavaScript modules, `import` must always be the first statement.
2. Loose `querySelectorAll` code was placed outside any function, running once on page load and hiding all tab content immediately.
3. `switchViewerTab` was reverted to the old version with the wrong guard clause.

**Fix:**

1. Moved `entityNavStack` declaration to below the `import` statement.
2. Deleted the two loose lines outside the function (they were already correctly inside `switchTab`).
3. Replaced `switchViewerTab` with the corrected version using `allEntities.length` as the guard.

**What I learned:** After merging a teammate's code, always check that your own work is still intact. Merge conflicts do not always produce visible errors. Sometimes code is silently overwritten or duplicated in the wrong place.

---

### Property existence checks must be at the right level

Checking a whole object tells you whether that object exists. It does not tell you whether the specific key you need is inside it.

```js
// WRONG: always truthy if allProperties exists at all, even if it has no data for this entity
if (window.state.allProperties) {
  showProperties(entityId);
}

// CORRECT: checks whether data exists for the specific entity
if (window.state.allProperties?.[entityId]) {
  showProperties(entityId);
}
```

This was the cause of entity links in the Relationships tab only showing the Identity section. The check was passing for every entity even when only Identity data had been loaded.

---

### Map lookups vs repeated .find() calls

Using `.find()` on an array inside a loop is O(n) every time it runs. Converting the array to a `Map` once gives O(1) lookups and is significantly faster when there are many entities to look up.

```js
// SLOW: .find() scans the whole array every time
function getEntity(id) {
  return window.state.allEntities.find((e) => e.id === id);
}

// FAST: build the Map once, then look up by key instantly
const entityMap = new Map(window.state.allEntities.map((e) => [e.id, e]));

function getEntity(id) {
  return entityMap.get(id);
}
```

---

### Vite tree-shaking and module registration

Vite strips imports that have no side effects during the build process. If you import a file just to run it (a side-effect-only import), Vite may remove it. Functions must be explicitly exported from their file and registered somewhere that Vite can trace as used.

```js
// WRONG: Vite may tree-shake this away because nothing is imported from it
await import('./js/ui-viewer.js');

// CORRECT: explicitly export functions from ui-viewer.js
export function toggleViewerCollapse() { ... }

// Then register them via window-api.js so they are reachable
import { toggleViewerCollapse } from './ui-viewer.js';
window.toggleViewerCollapse = toggleViewerCollapse;
```

---

### Duplicate local definitions override imports silently

After moving a function to a new file and importing it, if the original definition still exists in the old file it will silently override the imported version. There is no error or warning.

```js
// ui.js -- after refactor, this import is at the top
import { generateSummaryStats } from "./ui-display.js";

// If this original definition is still in ui.js below the import,
// it overrides the imported version silently
function generateSummaryStats() {
  // old version -- this wins, not the import
}
```

After moving a function, always delete the original from the old file before testing.

---

### Async functions and lazy loading timing

If a function calculates stats or displays data that depends on lazy-loaded state, it must `await` the loaders before running the calculation. Calling the function before the loaders finish results in zeros or missing data.

```js
// WRONG: calculates stats before data has loaded
function updateStatsOverview() {
  const count = window.state.allEntities.length; // 0 if not loaded yet
}

// CORRECT: await the loaders first, then calculate
async function updateStatsOverview() {
  await lazyLoadEntities(window.state.currentModelID);
  await lazyLoadProperties(window.state.currentModelID);
  const count = window.state.allEntities.length; // correct value now
}
```

---

## Resources

- [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [MDN -- Array.prototype.slice()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- [MDN -- Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- [MDN -- Math.max()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max)
- [MDN -- Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)
- [MDN -- setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout)
- [MDN -- Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [Vite -- Dependency Pre-Bundling](https://vitejs.dev/guide/dep-pre-bundling)
