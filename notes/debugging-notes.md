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

>

---

## Resources

- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Tricks Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
