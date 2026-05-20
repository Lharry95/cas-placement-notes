# Week 4 - Monday 11th May - 17th May 2026

## What I worked on

- Implemented pagination across multiple tabs in the IFC Parser (Entities, Properties, Quantities, Materials, Spatial, Summary)
- Built a reusable `paginate()` function in `ui.js` that handles page slicing, controls HTML, and "Showing X–Y of Z" info text
- Built a `goToPage()` function that updates `paginationState` and re-renders the correct tab using a `switch` statement
- Added page reset logic so that when a new IFC file is loaded, all tabs return to page 1 (`paginationState` reset in `displayResults()`)
- Fixed a bug where the Next button in pagination stopped working — caused by a missing closing quote on a CSS class name inside a template literal
- Fixed a `goToPage` bug where page numbers could go out of bounds when switching to a smaller file — solved using `Math.max(0, newPage)` (clamping)
- Removed an unused `onChange` parameter from `paginate()`
- Restructured the Entities tab to display entities grouped by IFC type (e.g. IfcWall, IfcDoor) using collapsible group headers with `toggleGroup()`
- Fixed a `groupId is not defined` bug in `displayEntitiesTab()`, a single missing `const groupId = \`group-${type}\`` line was breaking the whole function
- Made the Properties tab start collapsed by default, added `collapsed` to the collapsible header and `hidden` to the content div
- Investigated why Properties tab needs to filter entities that actually have properties before paginating. Understood that paginating unfiltered entities causes incorrect page counts due to `continue` skipping empty rows during render but not during counting

## What I learned

- How to build reusable pagination in Vanilla JavaScript, one `paginate()` function handles all tabs, each identified by a `tabKey` string
- How `Math.max(0, newPage)` is used as an industry-standard "clamp" to prevent page numbers going below zero
- Why grouping and filtering data _before_ paginating matters. Paginating unfiltered data with skipped rows causes wrong page counts and empty pages
- How a single missing variable declaration (`const groupId`) can silently break an entire render function
- How unclosed quote characters inside template literals cause cascading HTML parse failures that are hard to spot

## Challenges

- Tracking down the Next button bug — the missing `"` on `pagination-page` class name was a one-character bug that broke the entire button
- Understanding why `goToPage` needed bounds checking and what "clamping" meant in practice
- Keeping pagination consistent across six different tabs without duplicating logic

## Solutions

- Wrote `paginate()` once and called it from every display function with just two arguments, items array and tab key string
- Used `Math.max(0, newPage)` in `goToPage()` to clamp the lower bound, with button `disabled` attributes handling the upper bound
- Filtered entities down to only those with properties _before_ passing to `paginate()`, so the count and slicing are always accurate
- Fixed the Next button by replacing single quotes with double quotes and adding the missing closing `"` in the template literal

## Key takeaways

- Reusable functions with clear inputs and outputs (like `paginate(items, tabKey)`) keep code clean and consistent across many parts of an app
- Clamping numbers to a valid range is a standard practice, `Math.max` and `Math.min` are the tools for it
- One character bugs (a missing `"`) can cause symptoms that look much bigger than they are, read error messages carefully and check the HTML output
- Dead code (unused parameters, unused variables) should be removed as soon as it's identified to keep the codebase clean

## Reflection

- This week we heavily focused on JavaScript logic instead of visual styling, which was a different kind of challenge
- Building pagination from scratch gave me a much better understanding of how data slicing, state, and re-rendering work together
- I want to keep getting more comfortable tracing through JavaScript logic to find bugs rather than guessing at the cause
