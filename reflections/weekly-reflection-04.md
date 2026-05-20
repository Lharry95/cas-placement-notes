# Weekly Reflection - Week 4

## Wins

- Successfully built a reusable pagination system across six tabs using a single `paginate()` function
- Tracked down and fixed a one-character bug (missing `"`) that was breaking the Next button
- Understood and applied `Math.max(0, newPage)` clamping which is an industry-standard fix for out-of-bounds page numbers
- Grouped the Entities tab by IFC type using collapsible headers
- Removed dead code (`onChange` parameter) and kept the codebase clean

## Challenges

- Working through JavaScript logic rather than visual styling which a different kind of thinking is required
- Diagnosing the Next button bug, having to find exactly where the typo was.
- Understanding _why_ data needs to be filtered before paginating, not just _how_ to do it

## Key lessons

- A single reusable function (`paginate(items, tabKey)`) is a lot better than writing the same pagination logic six times
- Clamping (`Math.max`, `Math.min`) is the standard way to keep numbers inside a valid range, it's used in game engines, carousels, scroll handlers, and everywhere else
- You must filter or prepare your data _before_ paginating, pagination counts what it receives, it doesn't know about rows being skipped during render
- One-character bugs can produce big symptoms, you have to read the actual HTML output in DevTools when something renders wrong
- Removing unused parameters and dead code is part of good development practice, not just cosmetic tidying

## What I want to improve

- Getting faster at reading JavaScript errors and tracing them back to the source
- Understanding more about how state management works in Vanilla JS vs frameworks like React
- Building more confidence working with data transformations (filtering, grouping, slicing) before rendering

## Next week's focus

- Add error messages so the app tells users clearly when something goes wrong
- Add loading skeletons (placeholder animations) so panels look tidy while data is still loading
- Change colours to stand out more
