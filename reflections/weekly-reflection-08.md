# Weekly Reflection - Week 8

## Wins

- Resolved a major post-merge regression that was breaking six tabs at once, tracing it through three separate root causes across multiple files
- Fixed the 3D viewer lag by identifying and removing a redundant index builder that was blocking the main thread
- Fixed the quantities empty values bug by identifying that the chunked cache was using `WriteLine` instead of `GetLine`
- Kept all of Zeus's code completely untouched by applying every fix at the caller level
- Addressed all outstanding GitHub Copilot PR review comments

## Challenges

- The post-merge regression had no obvious error messages. Empty tabs and missing data required systematic investigation across multiple files to find the root cause
- Understanding the `GetLine` vs `WriteLine` distinction in web-ifc and why using the wrong one fails silently with no error thrown
- Working through multiple layers of the same problem in the correct order without introducing new regressions

## Key lessons

- When a new parser replaces an existing one, all downstream consumers must be audited for the specific keys they depend on before the merge goes in
- Never modify a teammate's file to fix an interface mismatch. Fix it at the point where you consume their output
- `GetLine` reads a line from the IFC model. `WriteLine` writes or modifies it. They are not interchangeable and using the wrong one returns empty data with no error thrown
- Expensive operations on the main thread at file load time block the renderer. Keep the load path lean and defer heavy work to lazy loaders
- Always check for duplicate function definitions after a merge. A merge can silently replace a working function with an older version
- Regression testing after a merge must cover every tab and feature, not just the ones the merge was supposed to affect

## What I want to improve

- Getting faster at systematically tracing regressions through a multi-file codebase
- Building a better mental model of the web-ifc API so I can spot potential method misuse earlier
- Reviewing merge diffs more carefully before merging so regressions are caught before they reach master

## Next week's focus

- Attend the handover
- Be ready to answer questions about the file structure, known bugs, and anything still outstanding
- Take on feedback and fix whatever needs fixing before full handover is completed.
