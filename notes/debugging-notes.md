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

## Week 7

### Data Loading But Not Displaying After Refactor

**What happened:** After the `ui.js` file separation refactor, loading an IFC file showed all the correct console log messages but nothing appeared in any of the tabs.

**Root cause:** `generateSummaryStats` and `generateMaterialTakeOff` had been moved to `ui-display.js` and imported at the top of `ui.js`, but the original definitions were still sitting below in `ui.js` and silently overriding the imports.

**Fix:** Deleted the two duplicate function definitions from `ui.js`.

**What I learned:** Moving a function to a new file is only half the job. The original must be deleted or it will silently override the import. Always search the old file for the function name after moving it.

---

### Entity Links in Relationships Tab Only Showing Identity Section

**What happened:** Clicking an entity link navigated to the entity detail view but only the Identity section had data.

**Root cause:** The condition checking whether property data had loaded was testing `window.state.allProperties` as a whole object. Because the object existed, the check always passed even when the specific entity's data had not been loaded.

**Fix:** Changed the check to test for the specific entity key.

```js
// WRONG: passes as long as allProperties exists
if (window.state.allProperties) { ... }

// CORRECT: checks whether data exists for this specific entity
if (window.state.allProperties?.[entityId]) { ... }
```

**What I learned:** A check at the wrong level of a data structure will pass when it should not. Always check the specific key you actually need.

---

### Stats Bar Showing Zeros After File Load

**What happened:** After loading an IFC file, the stats bar showed zeros for all counts.

**Root cause:** `updateStatsOverview` was being called before `lazyLoadEntities` and `lazyLoadProperties` had finished populating state.

**Fix:** Made `updateStatsOverview` async and added `await` calls for both lazy loaders before the stats calculation runs.

```js
async function updateStatsOverview() {
  await lazyLoadEntities(window.state.currentModelID);
  await lazyLoadProperties(window.state.currentModelID);
  const count = window.state.allEntities.length;
}
```

**What I learned:** Any function that depends on lazy-loaded data must await the loaders before running.

---

### Vite HMR WebSocket Error on Dev Server Start

**What happened:** The Vite development server was throwing a WebSocket connection error on startup, disrupting Hot Module Replacement.

**Root cause:** No `server.hmr` configuration was set in `vite.config.js`.

**Fix:** Added a `server.hmr` block to `vite.config.js` with the correct host and port settings.

**What I learned:** Vite HMR relies on a WebSocket connection between the browser and the dev server. Configuring `server.hmr` explicitly fixes environment-specific connection issues.

---

### Module Import Failure From Missing Closing Brace

**What happened:** The entire module failed to import and threw a syntax error on load.

**Root cause:** A missing closing brace `}` in an `if` block in `parser.js` made the file unparseable.

**Fix:** Added the missing `}` in the correct place.

**What I learned:** A single missing brace can make an entire file fail to parse. When a module import fails with a syntax error and nothing obvious is wrong, check for unclosed blocks by working backwards from the end of the file.

---

## Week 8

### Post-Merge Regression Across Six Tabs

**What happened:** After merging the Zeus branch, the Quantities, Materials, Project Info, Units, Footprints, and Floor Plan tabs all stopped displaying data.

**Root cause:** `buildIfcIndexesChunked` in Zeus's `chunked-parser.js` returned only `byType` and `byGUID` indexes. The existing `parser-properties.js` and `parser-footprints.js` also required `relDefinesByObject` and `relSpaceBoundaryBySpace`. Those keys did not exist in the chunked version, so every lookup silently returned nothing.

**Fix:** In `file-handler.js`, after `buildIfcIndexesChunked` ran, the two missing keys were added by calling `buildIfcIndexes` and merging the result. Zeus's `chunked-parser.js` was not modified.

```js
window.state.ifcIndexes = await buildIfcIndexesChunked(
  window.state.currentModelID
);

const { relDefinesByObject, relSpaceBoundaryBySpace } = buildIfcIndexes(
  window.state.currentModelID
);
window.state.ifcIndexes.relDefinesByObject = relDefinesByObject;
window.state.ifcIndexes.relSpaceBoundaryBySpace = relSpaceBoundaryBySpace;
```

**What I learned:** When a new parser replaces an existing one, all downstream consumers must be audited for the specific keys they depend on. Index shape mismatches fail silently with no error thrown. Never modify a teammate's file to fix an interface mismatch. Fix it where you consume their output.

---

### Project Info and Units Tabs Empty After Merge

**What happened:** Even after fixing the index mismatch, the Project Info and Units tabs were still empty.

**Root cause:** `extractModelInfoChunked` returned placeholder data for units and project info rather than real values from the IFC file.

**Fix:** After `extractModelInfoChunked` ran, the real `extractModelInfo` was called to supplement the result with actual data.

**What I learned:** A performance-optimised version of a function may not extract the same data as the original. When replacing a function with an optimised version, check that the output is equivalent, not just structurally similar.

---

### Quantities Tab Showing Empty Name and Value

**What happened:** The Quantities tab was rendering rows correctly but every row had `name: ""` and `value: null`.

**Root cause:** `getQuantitiesSmart` in `parser-properties.js` uses `window.state.ifcLineCache.get()` to fetch quantity data. The chunked cache had a `.get()` method that called `window.state.ifcApi.WriteLine` internally instead of `GetLine`. `WriteLine` is a write method, not a read method, so the data returned was empty.

**Fix:** Updated `lazy-loader.js` so that when properties load, `ifcLineCache` is replaced with a cache that uses `GetLine` correctly.

**What I learned:** In the web-ifc API, `GetLine` reads a line from the model and `WriteLine` writes or modifies it. They are not interchangeable. Using the wrong one returns empty data with no error thrown.

---

### 3D Viewer Lagging After Merge

**What happened:** After the merge, the 3D viewer became noticeably laggy when loading a file.

**Root cause:** `file-handler.js` was calling both `buildIfcIndexesChunked` and `buildIfcIndexes` back to back on the main thread at file load time. The second call was a full synchronous scan of every relationship in the IFC file, blocking the renderer.

**Fix:** Removed the `buildIfcIndexesChunked` call entirely and used only `buildIfcIndexes`.

```js
// Before: two index builders running back to back
window.state.ifcIndexes = await buildIfcIndexesChunked(
  window.state.currentModelID
);
const { relDefinesByObject, relSpaceBoundaryBySpace } = buildIfcIndexes(
  window.state.currentModelID
);

// After: one builder that produces everything needed
window.state.ifcIndexes = buildIfcIndexes(window.state.currentModelID);
```

**What I learned:** Running expensive operations on the main thread at file load time blocks the renderer. Running the same work twice is always a sign something went wrong during a merge.

---

## Debugging Checklist

When something is not working, go through this list:

- [ ] Check the browser console for errors
- [ ] `console.log()` the value you are unsure about
- [ ] Check if the same code exists in more than one place (HTML and JS)
- [ ] Check for duplicate element IDs in the HTML
- [ ] Check what CSS classes are applied to an element on load
- [ ] Check if the element exists in the DOM before selecting it
- [ ] Inspect the actual rendered HTML in DevTools. The cause is often smaller than the symptom
- [ ] Check that variable names inside functions are not accidentally in quotes
- [ ] Check that variables used in `catch` blocks are declared inside that block
- [ ] For objects, use `Object.keys(obj).length === 0` not `!obj` to check if empty
- [ ] For async data, make sure guard clauses run after the `await`, not before it
- [ ] After merging a teammate's code, check that your own work is still intact
- [ ] After moving a function to a new file, delete the original from the old file
- [ ] Check that property existence checks are at the right level, not one level too high
- [ ] If data is loading but not displaying, check for duplicate function definitions overriding imports
- [ ] After a merge, audit all downstream consumers of any replaced function for the specific keys they depend on
- [ ] In web-ifc, check whether a cache abstraction is wrapping `GetLine` or `WriteLine`
- [ ] Read the error message carefully. It usually tells you where the problem is
- [ ] If the bug is too large to solve alone, document it clearly and escalate

---

## Resources

- [Chrome DevTools Docs](https://developer.chrome.com/docs/devtools/)
- [MDN Debugging HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Debugging_HTML)
- [MDN try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)
- [MDN Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)
- [MDN setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout)
- [Vite Dependency Pre-Bundling](https://vitejs.dev/guide/dep-pre-bundling)
- [web-ifc API docs](https://ifcjs.github.io/info/docs/Guide/web-ifc/Introduction)
