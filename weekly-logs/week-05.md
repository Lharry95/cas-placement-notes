# Week 5 - Monday 18th May - 24th May 2026

## What I worked on

- Added error handling across all UI display functions in `ui.js`
- Wrote guard clauses for `displayPropertiesTab`, `displayMaterialsTab`, `displayQuantitiesTab`, `displaySpatialTab`, `displayFootprintsTab`, `displayRawTab`, and `showEntityDetail`
- Fixed the `lazyLoadTabData` catch block to dynamically target the correct tab container using `tabName + "Content"` — avoiding repetition across six functions
- Wrote the `.error-banner` CSS class to display user-friendly red error messages inside tab panels
- Built loading skeleton animations for the Entities, Quantities, and Materials tabs
- Wrote `@keyframes shimmer`, `.skeleton`, `.skeleton-row-header`, and `.skeleton-row-item` CSS classes
- Built the `showSkeleton()` JavaScript function that injects placeholder rows into a tab container before data loads
- Wired `showSkeleton()` into the `lazyLoadTabData` switch statement — called before each `await` for entities, quantities, and materials
- Fixed a bug in the catch block where `container` was being referenced before it was declared
- Fixed incorrect empty checks on object types — learned that `Object.keys().length === 0` is the correct way to check if an object is empty

## What I learned

- **Error handling has two audiences**: users see friendly plain-English messages written to `container.innerHTML`; developers see specific technical messages via `console.error`
- **Guard clause order** always: define container → guard container → guard `window.state` → guard specific data → guard empty results
- **Defensive programming**: each guard sits immediately before the line that needs it, not all bunched at the top
- **DRY principle**: `tabName + "Content"` targets any tab's container dynamically with one line instead of six separate lines
- **What a loading skeleton is**: a placeholder animation that mirrors the shape of real content while data loads, making the app feel faster even if load time hasn't changed (perceived performance)
- **The three rules of skeletons**: mirror the real layout, animate to signal activity, swap out when data arrives
- **CSS animation timing**: `@keyframes` defines the timeline, `1.5s` is one complete cycle, `infinite` repeats forever, `ease-in-out` controls the smoothness
- **Viewport filling**: show enough skeleton rows to fill the visible screen, not the full dataset
- **Collapsed state awareness**: skeletons mirror what users see first; since groups start collapsed, only header rows need skeletons
- **Checking if an object is empty**: `Object.keys(obj).length === 0` is the correct check; `!obj` only catches null/undefined

## Challenges

- Understanding where to place `showSkeleton()` in the `lazyLoadTabData` function, the key realisation was it needed to go _before_ the `await`, not after it
- Distinguishing between a string in quotes (`"containerID"`) and a variable without quotes (`containerId`) inside a function — using the string would search for a literal element that doesn't exist
- Fixing the catch block bug where `container` was referenced before being declared, the fix was to declare it inside the catch, not reference it from outside
- Understanding why `!obj` is not enough to check if an object is empty, an empty object `{}` is still truthy in JavaScript

## Solutions

- Placed `showSkeleton()` before the `await` so the skeleton appears while the app is waiting, not after the data has already arrived
- Removed quotes from `containerId` inside `showSkeleton()` so it reads the variable passed in, not a literal string
- Moved the `container` declaration inside the catch block so it's always defined when needed
- Used `Object.keys(window.state.modelInfo.project).length === 0` to correctly detect an empty project object

## Key takeaways

- Error handling has two separate goals, keeping the user informed and helping the developer debug
- Guard clauses should be layered from outermost to innermost, like checking the box exists before checking what's inside it
- Perceived performance matters, users feel better when something is visible during loading, even if load time is the same
- Sprint scope is important, skeletons were added only to tabs within scope (entities, quantities, materials), not teammates' code
- One character mistakes (quotes vs no quotes) can cause silent bugs that are easy to miss

## Reflection

- This week felt different to previous weeks, less about visual design and more about making the app reliable and user-friendly in ways that aren't immediately visible
- Error handling and loading states are things users notice when they're missing, not when they're there which makes them easy to skip but important to build
- I want to get more comfortable reasoning through JavaScript logic before writing code, rather than writing and then fixing
