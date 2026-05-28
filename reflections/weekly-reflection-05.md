# Weekly Reflection - Week 5

## Wins

- Successfully added production-quality error handling across all eight major UI display functions
- Built loading skeleton animations from scratch, `@keyframes`, CSS classes, and the `showSkeleton()` function
- Wired skeletons into the lazy loading system correctly, appearing before data loads, disappearing when it arrives
- Fixed three separate bugs during the session (catch block reference error, incorrect empty object check, missing container guard)
- Understood and applied the DRY principle, one dynamic line replacing six repetitive ones

## Challenges

- Knowing _where_ to place `showSkeleton()`. Had to think through the timing carefully before the placement made sense
- Distinguishing between a quoted string and a variable, a one-character difference that causes a silent bug
- Understanding why checking `!obj` isn't enough for empty objects in JavaScript, empty objects are still truthy
- Shifting mindset from visual work to reliability work, error handling and skeletons are less visible but just as important

## Key lessons

- Error handling always has two audiences, users get plain English, developers get `console.error`
- Guard clause order is always the same: container → state → specific data → empty check
- Perceived performance matters, users feel faster load times when something is visible, even if the actual time hasn't changed
- Collapsed state awareness, skeletons should only show what users can see first, not hidden content inside collapsed groups
- Sprint scope discipline, only touch your own lane; don't add skeletons to teammates' tabs without discussion
- `Object.keys(obj).length === 0` is the correct way to check if an object is empty in JavaScript

## What I want to improve

- Reasoning through JavaScript logic before writing it, less writing and fixing, more thinking first
- Getting more comfortable reading and understanding JavaScript timing (async, await, when things happen)
- Applying consistent patterns across the codebase without needing to be reminded each time

## Next week's focus

- Make the 3D viewer collapsible — add a toggle bar so the canvas can be collapsed and expanded
- Make the left sidebar collapsible — shrink to icon-only strip when toggled
- Add bottom tabs to the 3D viewer panel (Quantities, Units, Materials, Entities) so data is accessible alongside the model
- Wire up `toggleViewerCollapse()` and `toggleLeftSidebar()` functions in `ui.js`
- Use CSS variables (`--sidebar-width`) to control grid layout width dynamically from JavaScript
