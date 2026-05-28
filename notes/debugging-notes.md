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

**Fix:** Still unresolved.

**What I learned:** Panel-based navigation needs to track state so users can move backwards through views. This is different from browser navigation and needs to be handled manually in JavaScript.

---

## Week 4

### Pagination Next Button Stopped Working

**What happened:** The Next button in the pagination controls stopped working entirely after it was added.

**Root cause:** A missing closing `"` on the `pagination-page` CSS class name inside a template literal. One character caused the entire button's HTML to be malformed.

**Fix:** Added the missing closing quote in the template literal.

```js
// BROKEN: missing closing " on the class name
`<span class="pagination-page>Page ${n}</span>`// FIXED
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

**Root cause:** A single missing variable declaration `const groupId = \`group-${type}\``inside`displayEntitiesTab()`. Every line referencing `groupId` after it was breaking.

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

## Debugging Checklist

When something isn't working, go through this list:

- [ ] Check the browser console for errors
- [ ] `console.log()` the value you're unsure about
- [ ] Check if the same code exists in more than one place (HTML and JS)
- [ ] Check what CSS classes are applied to an element on load
- [ ] Check if the element exists in the DOM before selecting it
- [ ] Inspect the actual rendered HTML in DevTools. The cause is often smaller than the symptom
- [ ] Check that variable names inside functions aren't accidentally in quotes
- [ ] Check that variables used in `catch` blocks are declared inside that block
- [ ] For objects, use `Object.keys(obj).length === 0` not `!obj` to check if empty
- [ ] Read the error message carefully. It usually tells you where the problem is
- [ ] If the bug is too large to solve alone, document it clearly and escalate

---

## Resources

- [Chrome DevTools Docs](https://developer.chrome.com/docs/devtools/)
- [MDN – Debugging HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Debugging_HTML)
- [MDN – try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)
- [MDN – Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)
