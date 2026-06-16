# Week 8 - Monday 8th June - 14th June 2026

## What I worked on

- Merged the Zeus branch into master and resolved Git merge conflicts across `file-handler.js`, `ui.js`, and `package-lock.json`
- Diagnosed a major post-merge regression where Quantities, Materials, Project Info, Units, Footprints, and Floor Plan tabs all stopped displaying data
- Traced the regression to a mismatch between `buildIfcIndexesChunked` (from Zeus's `chunked-parser.js`) and the existing `buildIfcIndexes` function. The chunked version only returned `byType` and `byGUID` indexes but `parser-properties.js` and `parser-footprints.js` also required `relDefinesByObject` and `relSpaceBoundaryBySpace`
- Fixed the index mismatch in `file-handler.js` by merging both index shapes together after the chunked version ran, without modifying Zeus's code
- Fixed a second issue where `extractModelInfoChunked` was returning placeholder data for units and project info, supplemented by calling the real `extractModelInfo` function
- Diagnosed and fixed the quantities tab showing empty `name: ""` and `value: null` values. The cause was `getQuantitiesSmart` using `ifcLineCache.get()` from the chunked cache which was calling `WriteLine` instead of `GetLine`, returning empty data. Fixed in `lazy-loader.js`
- Fixed 3D viewer lag introduced by the merge. The cause was `buildIfcIndexesChunked` and `buildIfcIndexes` both running back to back on the main thread at file load time. Fixed by removing the redundant chunked call and using only `buildIfcIndexes`
- Addressed GitHub Copilot PR review comments across `ui.js` and `parser.js` including null guards, `aria-expanded` placement, removing a broken `showSkeleton("generalInfo")` call, and cleaning up a duplicate `<span>` block
- Fixed a syntax error from a missing closing brace in an `if` block that was causing a module import failure
- Fixed a duplicate `switchTab` function left over from the merge that was silently overriding the correct version
- Sent a plain-language team summary message explaining what broke, why it broke, what was fixed, and what was not touched

## What I learned

- **Index shape mismatches are silent failures.** When a new chunked parser replaced an existing one, the downstream consumers that depended on specific index keys got nothing back with no error thrown. All downstream consumers must be audited whenever a parser is replaced
- **Fix interface mismatches at the caller, not the source.** Rather than modifying Zeus's `chunked-parser.js`, the fix was applied in `file-handler.js` where the indexes are assigned. This respects the principle of not modifying code you do not own
- **`GetLine` reads, `WriteLine` writes.** In the web-ifc API these are not interchangeable. Using `WriteLine` where `GetLine` is needed returns empty data silently with no error thrown
- **Running two index builders back to back blocks the renderer.** Expensive synchronous operations on the main thread at file load time freeze the 3D viewer. The file load path should be kept lean and heavy work deferred to lazy loaders
- **A duplicate function definition from a merge overrides silently.** When a duplicate `switchTab` came in via the merge it replaced the working version with no error. The symptom was wrong behaviour, not a crash
- **Regression testing after a merge is not optional.** Every tab and feature needs to be checked after pulling teammate code, not just the features the merge was supposed to affect

## Challenges

- The post-merge regression had no obvious error messages. Empty tabs required systematic file-by-file investigation to find the root cause across three separate issues
- Understanding the `GetLine` vs `WriteLine` distinction in web-ifc and why using the wrong one fails silently
- Working through multiple layers of the same regression in the correct order without introducing new issues along the way

## Solutions

- Audited every downstream consumer of `window.state.ifcIndexes` to find all the keys they depended on, then merged the missing keys in `file-handler.js`
- Left Zeus's `chunked-parser.js` completely untouched and applied all fixes at the caller level
- Removed the redundant `buildIfcIndexesChunked` call so only `buildIfcIndexes` runs at file load time
- Worked through Copilot PR comments one at a time, verifying each fix before moving to the next

## Key takeaways

- When a new parser replaces an existing one, audit all downstream consumers for the specific keys they depend on before merging
- Never modify a teammate's file to fix an interface mismatch. Fix it at the point where you consume their output
- `GetLine` reads a line from the IFC model. `WriteLine` writes or modifies it. They are not interchangeable
- Running expensive operations on the main thread at file load time blocks the renderer. Always check what is happening at load time after a merge
- Always check for duplicate function definitions after a merge

## Reflection

- This week was the most intense debugging session of the placement, working through a layered regression across multiple files all introduced by a single merge
- It reinforced how important it is to understand what a merge actually changes before assuming it is safe to ship
- The project is at handover and the codebase is in a significantly better state than it was at the start of placement
