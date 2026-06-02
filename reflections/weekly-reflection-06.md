# Weekly Reflection - Week 6

## Wins

- Built four separate features that all work together: collapsible sidebar, collapsible 3D viewer, bottom viewer tabs, and the entities back button
- Created a back button for entities
- Fixed six separate bugs during the week including a silent duplicate ID failure and a merge conflict with a teammate's code
- Understood and applied `setTimeout(fn, 0)` to control when JavaScript runs relative to the browser paint cycle
- Successfully diagnosed and fixed three post-merge breakages without losing any of the original work

## Challenges

- The 3D viewer going blank after restructuring the HTML a duplicate ID caused Three.js to target the wrong element with no error thrown
- Understanding why `height` transitions do not work on flex children and finding the correct fix with `max-height`
- The merge conflict a teammate's code reverted three things we had built and introduced a syntax error by placing code before an import statement
- Knowing which piece of state to guard against checking `currentModelID` instead of `allEntities.length` caused the tabs to show the wrong message at the wrong time

## Key lessons

- CSS Grid `1fr` automatically fills available space when neighbouring columns shrink no manual width calculations needed
- Use `max-height` not `height` for CSS transitions on flex children
- `setTimeout(fn, 0)` is a standard technique for yielding to the browser render thread so UI updates paint before heavy work runs
- Module-scope variables persist across function calls. Variables declared inside a function reset every time it runs
- Duplicate element IDs cause silent failures with no error thrown always keep IDs unique
- After merging a teammate's code, always check that your own work is still intact
- Guard clauses must check the right piece of state the wrong guard passes or fails at the wrong time

## What I want to improve

- Reading merge diffs more carefully so I can spot conflicts before things break
- Getting more comfortable reasoning through async timing: what runs first, what waits, what yields
- Understanding more about how CSS transitions work inside different layout contexts (flexbox vs grid vs block)

## Next week's focus

- Polish and test all six features end to end with a real IFC file
- Review and clean up any remaining inline styles or inconsistent class names
- Look at whether the bottom viewer tabs should be collapsible as well.
- Continue working on the relationships panel
