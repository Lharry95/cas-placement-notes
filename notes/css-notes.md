# CSS Notes

> Key concepts and things I've learned about CSS during placement

---

## The Box Model

Every element is a box made of:

- **Content** – the actual text/image
- **Padding** – space inside the border
- **Border** – the border itself
- **Margin** – space outside the border

```css
box-sizing: border-box; /* padding + border included in element's width */
```

---

## Flexbox Cheatsheet

```css
display: flex;
flex-direction: row | column;
justify-content: flex-start | center | flex-end | space-between | space-around;
align-items: flex-start | center | flex-end | stretch;
gap: 1rem;
```

---

## CSS Grid Basics

```css
display: grid;
grid-template-columns: repeat(3, 1fr); /* 3 equal columns */
gap: 1rem;
```

---

## Notes from Placement

### Changing Grid Column Layout

During Week 3, the IFC Parser layout was changed from a three-column grid to a two-column grid to better suit the UI.

```css
/* Before — three equal columns */
grid-template-columns: repeat(3, 1fr);

/* After — two equal columns */
grid-template-columns: repeat(2, 1fr);
```

The `repeat()` function takes two arguments — how many columns you want, and how wide each one should be. `1fr` means each column takes an equal share of the available space.

---

### Controlling Default Panel State with Class Names

CSS classes can be used to set the starting state of a component on page load — JavaScript only needs to toggle them from there.

```html
<div class="property-set collapsed">
  <div class="collapsible-content hidden">
    <!-- content starts hidden, JS toggles it open on click -->
  </div>
</div>
```

```css
.hidden {
  display: none;
}

.collapsed .collapsible-content {
  display: none;
}
```

---

### Font Size and Colour for Contrast

Adjusted font sizes and colours across the UI to improve readability and contrast between elements.

```css
/* Adjusting font size */
font-size: 14px; /* smaller for labels or secondary text */
font-size: 16px; /* base readable size for body text */
font-size: 1.2rem; /* relative to root — scales better */

/* Improving contrast */
color: #ffffff; /* light text */
background-color: #1e1e1e; /* dark background */

/* Check contrast ratio — aim for at least 4.5:1 for normal text */
```

---

### Font Size and Colour for Contrast

```css
font-size: 14px;
font-size: 1.2rem;
color: #ffffff;
background-color: #1e1e1e;
```

---

### Error Banner — user-facing error messages

A styled container for showing plain-English error messages inside tab panels when something goes wrong.

```css
.error-banner {
  background: rgba(245, 87, 108, 0.15);
  border: 1px solid rgba(245, 87, 108, 0.3);
  color: #f5576c;
  padding: 16px;
  border-radius: 8px;
  margin-top: 20px;
}
```

Used in HTML like this:

```html
<p class="error-banner">No data found. Please reload the file.</p>
```

The colour (`#f5576c`) is a red from the existing design system — consistent with other error states in the app. The low-opacity background keeps it subtle rather than alarming.

---

### Loading Skeleton Animation

A shimmer animation used to show placeholder content while real data is loading. Makes the app feel faster even if load time hasn't changed (perceived performance).

```css
/* The animation — fades in and out repeatedly */
@keyframes shimmer {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.4;
  }
  100% {
    opacity: 1;
  }
}

/* Base skeleton class — applied to every placeholder bar */
.skeleton {
  background: rgba(30, 30, 50, 0.6);
  border-radius: 4px;
  animation: shimmer 1.5s ease-in-out infinite;
}

/* Group header row — shorter, taller */
.skeleton-row-header {
  width: 25%;
  height: 25px;
  margin-bottom: 20px;
}

/* Item row — wider, shorter, indented */
.skeleton-row-item {
  width: 40%;
  height: 16px;
  margin-left: 15px;
  margin-bottom: 20px;
}
```

**Key decisions:**

- `rgba(30, 30, 50, 0.6)` — already in the design system, sits naturally on the dark background
- `1.5s` — one complete shimmer cycle (visible → faded → visible)
- `infinite` — keeps looping until real content replaces it
- `ease-in-out` — smooth transition, not a sharp flash
- Widths in `%` not `px` — scales across different screen widths
- Header row is narrower (25%) because IFC type names are short (e.g. "IfcWall")
- Item row is wider (40%) because entity names are longer

---

### CSS Grid and Collapsible Sidebars

When a sidebar is inside a CSS Grid layout, changing the element's `width` alone does not affect the grid column it sits in. The grid column itself must be updated via JavaScript for the neighbouring content to expand and fill the space.

```js
// WRONG: only shrinks the sidebar visually, does not free up grid space
sidebar.style.width = "48px";

// CORRECT: updates the grid column so the main content expands automatically
document.querySelector(".app-layout").style.gridTemplateColumns = "48px 1fr";
```

When the main content column is set to `1fr`, it automatically takes all remaining available space. You do not need to calculate a new pixel value.

---

### max-height Transitions on Flex Children

`height` transitions do not work reliably on elements inside a flexbox layout. Use `max-height` instead.

```css
/* Set a large max-height in CSS to allow the element to show fully */
#viewer-body {
  max-height: 2000px;
  transition: max-height 0.3s ease;
  overflow: hidden;
}
```

```js
// Collapse: set max-height to 0
viewerBody.style.maxHeight = "0px";

// Expand: restore the large value
viewerBody.style.maxHeight = "2000px";
```

The transition animates between the two values. This works because `max-height` has a concrete start and end value to animate between. `height: auto` does not the browser cannot calculate how to animate to an unknown value.

---

## Resources

- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Tricks Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [CSS Grid Guide CSS Tricks](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [MDN @keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes)
- [MDN max-height](https://developer.mozilla.org/en-US/docs/Web/CSS/max-height)
- [Contrast Checker WebAIM](https://webaim.org/resources/contrastchecker/)
