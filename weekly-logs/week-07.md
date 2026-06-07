# Week 7 - Monday 1st June - 7th June 2026

## What I worked on

- Resolved a series of bugs across `ui.js` and `parser.js` surfaced by GitHub Copilot PR review comments and merge conflicts
- Fixed a Git merge conflict affecting font sizes inside `buildNode` and resolved a `package-lock.json` conflict caused by a libc/musl vs glibc environment difference
- Fixed a syntax error caused by a duplicate `</tr>` closing tag in a table
- Removed a duplicate `switchTab` function left over from a merge that was silently overriding the correct version
- Fixed a missing closing brace in an `if` block that was causing a module import failure
- Replaced repeated `.find()` calls with a `Map` lookup for performance in `displayRelationshipsTab`
- Fixed `toggleViewerDataPanel` to use the correct variable name (`isDataPanelOpen` instead of `isOpen`)
- Added null guards, fixed `aria-expanded` placement, and removed a broken `showSkeleton("generalInfo")` call
- Cleaned up a duplicate `<span>` block in the viewer data panel toggle HTML
- Fixed `showEntityDetail` by making it `async` so it correctly lazy-loads properties when navigating to an entity from the Relationships tab
- Fixed a bug where entity links in the Relationships tab were only showing the Identity section. The cause was checking `window.state.allProperties` as a whole object instead of checking for the specific entity key `window.state.allProperties?.[entityId]`
- Fixed the stats bar showing zeros after file load. The cause was `updateStatsOverview` being called before lazy loaders had populated data. Fixed by making the function `async` and awaiting `lazyLoadEntities` and `lazyLoadProperties` before calculating counts
- Restored collapsed default state for Identity, Properties, Quantities, and Materials sections inside `showEntityDetail` after `display:none` had been incorrectly changed to `display:block` in a prior fix
- Resolved a Vite HMR WebSocket error by adding a `server.hmr` block to `vite.config.js`
- Completed a three-way file separation refactor of `ui.js` following the Single Responsibility Principle:
  - Created `ui-viewer.js` to hold all 3D viewer functions (`toggleViewerCollapse`, `toggleViewerDataPanel`, `switchViewerTab`, and the four `buildViewerXxxHTML` helpers)
  - Created `ui-display.js` to hold all tab display functions and stat helpers (`generateSummaryStats`, `generateMaterialTakeOff`, and all `displayXxxTab` functions)
  - Left `ui.js` as the clean coordinator: tab routing, skeleton loading, app triggers, UI toggles, and entity navigation
- Fixed a post-refactor bug where data was loading but not displaying. The cause was duplicate local definitions of `generateSummaryStats` and `generateMaterialTakeOff` still sitting in `ui.js` and silently overriding the imported versions from `ui-display.js`
- Ran a full regression test after the refactor to confirm all tabs, toggles, entity navigation, relationship links, and back button behaviour were working correctly
- Wrote a PR description and a plain-language explanation of the new file structure for non-technical teammates

## What I learned

- **Vite tree-shaking**: Vite strips side-effect-only imports during the build process. Functions must be explicitly exported and registered (e.g. via `window-api.js`) rather than relying on a bare `await import()` call in `main.js`
- **Duplicate local definitions override imports silently**: After moving a function to a new file and importing it, if the original definition still exists in the old file it will silently override the imported version. Originals must be carefully deleted after each move
- **Property existence checks must be specific**: Checking `window.state.allProperties` as a whole object is always truthy if the object exists at all. The correct check is `window.state.allProperties?.[entityId]` to confirm that the specific entity's data has been loaded
- **Lazy loading timing**: Stats and property data must be awaited before calculations run. Calling `updateStatsOverview` before `lazyLoadEntities` and `lazyLoadProperties` have finished results in zeros or missing data
- **Single Responsibility Principle in practice**: One file should do one thing. `ui.js` was doing three things (viewer logic, display logic, and coordination), so splitting it made each file easier to read, test, and change independently
- **Agile thinking about refactoring**: Splitting files further than needed adds complexity without solving a real problem. The right time to split again is when someone struggles to find something, or when the file grows significantly. Over-engineering is a risk
- **Map lookups vs repeated .find() calls**: Repeated `.find()` on an array is O(n) every time. Converting the array to a `Map` once and looking up by key is O(1) and much faster when there are many lookups
- **Regression testing after a refactor**: After any significant structural change, every feature should be tested end to end to confirm nothing broke. This is standard agile practice before raising a PR

## Challenges

- Tracking down why data was loading but not displaying after the refactor. The cause was duplicate function definitions in `ui.js` that were invisible unless you knew to look for them
- Understanding Vite tree-shaking and why a bare `await import()` was not enough to register functions
- Working through multiple Copilot PR comments across two files at once without losing track of what had and had not been fixed
- The `showEntityDetail` property check bug. The symptom (only Identity section showing) gave no obvious hint that the root cause was a property existence check at the wrong level

## Solutions

- Deleted the duplicate `generateSummaryStats` and `generateMaterialTakeOff` definitions from `ui.js` after moving them to `ui-display.js`
- Registered `ui-viewer.js` functions explicitly via `window-api.js` instead of relying on a bare import
- Worked through Copilot PR comments one at a time, verifying each fix before moving to the next
- Changed the property check from `window.state.allProperties` to `window.state.allProperties?.[entityId]` so it tests the specific entity, not the whole object

## Key takeaways

- Vite is not just a dev server. It actively tree-shakes unused imports. This changes how you need to structure module registration
- Moving a function to a new file is only half the job. The other half is deleting the original so it does not silently override the import
- Checking the right level of a data structure matters. A check that is one level too high will always pass even when the specific data you need is missing
- Regression testing is not optional after a refactor. It is the step that confirms the refactor actually worked without breaking anything
- Agile means knowing when to stop refactoring. Good enough and working is better than perfect and delayed

## Reflection

- This week felt like the most technically complex so far, working across merge conflicts, PR comments, a major refactor, and several runtime bugs at the same time
- The file separation refactor was something that had been building for a few weeks and completing it felt like a significant milestone
- Writing the plain-language summary for teammates was a useful exercise. It forced me to understand the structure well enough to explain it simply
- I want to get more comfortable reading Vite build output and understanding what it is telling me before things break
