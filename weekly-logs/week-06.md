# Week 6 - Monday 25th May - 31st May 2026

## What I worked on

- Built a collapsible left sidebar with a labelled toggle button that shrinks to an icon-only strip when collapsed
- Built a collapsible 3D viewer canvas with a labelled toggle bar that keeps the status bar and bottom tabs visible even when the canvas is hidden
- Added a bottom tab bar inside the 3D viewer with four tabs: Quantities, Units, Materials, and Entities
- Each bottom tab lazy-loads data from already-parsed IFC state into a scrollable panel without leaving the 3D view
- Implemented a navigation history stack in the Entities tab so users can drill into an entity detail view and click a back button to return to the list
- Fixed the merge conflict after a teammate's code was merged and reverted some of the implemented functions
- Used `setTimeout(fn, 0)` to yield to the browser render thread so the "Loading..." state is visible before data builds run
- Updated `grid-template-columns` via JavaScript on every sidebar toggle so the main content column expands automatically using CSS Grid `1fr`

## What I learned

- **CSS Grid `1fr` responds to what is available** when the sidebar column shrank from 240px to 48px, the `1fr` column for the main content automatically filled the extra space. You do not need to manually calculate new widths, `1fr` means "take whatever is left"
- **`max-height` is more reliable than `height` for transitions on flex children** `height` transitions do not work reliably when an element is inside a flexbox layout. The pattern is: set a large `max-height` value in CSS and toggle it to 0 in JavaScript
- **`setTimeout(fn, 0)` yields to the render thread** this pushes heavy JavaScript work to after the browser has painted the current state. Without it, the browser tries to do both at the same time and the "Loading..." message never appears. This is called yielding to the render thread
- **Module-scope variables must live outside functions** if `entityNavStack` is declared inside a function, it resets to an empty array every time the function is called. It needs to live at the top level of the file so it persists across calls
- **Navigation stack pattern** an array used as a stack: `push` saves the current state before navigating forward, `pop` restores it when going back. This naturally supports multiple drill-down levels
- **Duplicate IDs cause silent failures** `getElementById` only finds the first element with a given ID. A second element with the same ID is ignored with no error thrown, causing bugs that are hard to trace
- **Separation of concerns in HTML structure** placing the viewer info bar and bottom tab bar outside `viewer-body` meant they stayed visible when the canvas collapsed. If they had been inside, collapsing the canvas would have hidden the tabs too
- **Guard clause order matters when working with async data** checking `currentModelID` as a guard was wrong because it is set by the 3D loader, not the IFC parser. The correct guard is `allEntities.length` because that confirms the parsed data is ready

## Challenges

- The 3D viewer canvas went blank after restructuring the HTML caused by two divs both using `id="viewer-container"`, making Three.js target the wrong one
- The viewer collapse toggle was not working `height` transitions do not work reliably on flex children, had to switch to `max-height`
- Bottom tabs were showing "Load IFC first" even after a file was loaded the guard was checking `currentModelID` instead of `allEntities.length`
- Bottom tabs required clicking another tab first before showing data the guard ran before `lazyLoadEntities` had a chance to populate state, fixed by moving the guard inside the `setTimeout` after the `await`
- `ReferenceError: isSidebarOpen` the `let isSidebarOpen` declaration had been deleted from the file, re-added at module scope outside the function
- Main content area was not expanding when the sidebar collapsed CSS `width` on the sidebar does not affect the grid column it sits in, had to update `grid-template-columns` via JavaScript instead
- After merging a teammate's code, three things broke: `entityNavStack` was declared before the `import` statement causing a syntax error, loose `querySelectorAll` code was running outside a function and hiding all tabs on load, and `switchViewerTab` was reverted to the old broken guard

## Solutions

- Removed the duplicate `id="viewer-container"` wrapper div so Three.js targets the correct element
- Switched from `height` to `max-height` transition on `#viewer-body`
- Changed the guard in `switchViewerTab` from `currentModelID` to `allEntities.length`
- Moved the guard clause to inside the `setTimeout` callback, after the `await lazyLoadEntities` call
- Re-added `let isSidebarOpen = true` at module scope at the top of `ui.js`
- Updated `grid-template-columns` on the layout element directly via JavaScript on every toggle
- Fixed the merge conflict by moving `entityNavStack` below the import, deleting the loose `querySelectorAll` lines, and replacing `switchViewerTab` with the corrected version

## Key takeaways

- `1fr` in CSS Grid automatically fills available space no manual width calculations needed
- Always use `max-height` for transitions on flex children, not `height`
- `setTimeout(fn, 0)` is a standard technique for letting the browser paint before running heavy work
- State variables that need to persist across function calls must be declared at module scope, not inside the function
- One element per ID duplicates cause silent failures that are hard to debug
- Guard clauses must check the right piece of state wrong guard means the check passes or fails at the wrong time
- Working in a shared codebase means merges can overwrite your work always review what changed after pulling a teammate's code

## Reflection

- This week felt like the most complete sprint so far four separate features all working together by the end
- Fixing the merge conflict was a good reminder that working in a team codebase means staying aware of what others are changing
- I want to get more comfortable reading merge diffs so I can spot conflicts earlier rather than discovering them when things break
