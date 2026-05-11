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

## Resources

- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Tricks Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [CSS Grid Guide – CSS Tricks](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Contrast Checker – WebAIM](https://webaim.org/resources/contrastchecker/)
