# Debugging Notes

> Key concepts and bugs I've encountered and solved during placement

---

## Duplicate Buttons Rendering on Page

**What happened:** The Footprints tab was showing double buttons — All Doors, Inward Only, Outward Only, and Unknown Direction were all appearing twice.

**Root cause:** The button code existed in both the HTML file and in `UI.JS`, causing them to render twice.

**Fix:** Removed the button code from the HTML and kept it only in `UI.JS`.

```js
// Keep button logic in UI.JS only — not in the HTML
// Defining buttons in both places causes them to render twice
```

**What I learned:** When UI elements are defined in both HTML and JavaScript, they can render twice. Logic like this belongs in one place — if it's being generated dynamically, keep it in JavaScript only.

---

## Collapsible Panels Starting Open Instead of Collapsed

**What happened:** Property set panels were loading in an open/expanded state by default, when they should start collapsed and only open when clicked.

**Root cause:** No default collapsed state was set on the HTML elements — the CSS classes controlling visibility weren't applied on load.

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

**What I learned:** Default UI state can be set through CSS class names directly in the HTML — JavaScript only needs to toggle the classes from there. You don't need JS to control what something looks like on first load.

---

## App Freezes When Large IFC File Is Loaded

**What happened:** When a large IFC file is loaded, only the 3D viewer displays correctly. Clicking any other tab (e.g. Entities) causes the application to freeze.

**Root cause:** Under investigation — likely a performance issue with how tab content is loaded alongside a large file.

**Fix:** Escalated to the team — not yet resolved.

**What I learned:** Large file handling may require performance optimisation such as lazy loading tab content rather than loading everything at once.

---

## No Back Button in Entities View

**What happened:** When navigating into a specific entity in the Entities view, there is no way to return to the entities list — no back button or navigation exists.

**Root cause:** Navigation state inside the panel isn't being tracked, so there's no history to go back to.

**Fix:** Still unresolved — to be investigated next week.

**What I learned:** Panel-based navigation needs to track state so users can move backwards through views. This is different from browser navigation — it needs to be handled manually in JavaScript.

---

## Debugging Checklist

When something isn't working, go through this list:

- [ ] Check the browser console for errors
- [ ] `console.log()` the value you're unsure about
- [ ] Check if the same code exists in more than one place (HTML and JS)
- [ ] Check what CSS classes are applied to an element on load
- [ ] Check if the element exists in the DOM before selecting it
- [ ] Read the error message carefully — it usually tells you where the problem is
- [ ] If the bug is too large to solve alone, document it clearly and escalate

---

## Resources

- [Chrome DevTools Docs](https://developer.chrome.com/docs/devtools/)
- [MDN – Debugging HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Debugging_HTML)
