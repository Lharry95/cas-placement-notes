# Weekly Reflection - Week 3

## Wins

- Successfully fixed default panel behaviour so collapsible sections start hidden on load
- Identified and resolved a duplicate buttons bug by consolidating code from HTML into `UI.JS`
- Continued building on the relationships panel with CSS styling and structure

## Challenges

- Diagnosing the freezing bug when large IFC files are loaded — complex enough to escalate to the team
- Understanding how navigation state should work inside the Entities view
- Working across multiple parts of the codebase (HTML, CSS, JS) simultaneously and keeping changes consistent

## Key lessons

- Default UI state can be controlled through CSS class names in HTML (e.g. `collapsed`, `hidden`) rather than relying on JavaScript to set it on load
- When the same logic exists in both HTML and JavaScript, it causes duplication — keeping it in one place (JS) is cleaner and easier to maintain
- Some bugs are beyond the scope of one developer to fix alone — knowing when to escalate is an important skill
- Small targeted fixes (like class names) can have a meaningful impact on user experience

## What I want to improve

- Understanding how to implement navigation history inside panels (back button functionality)
- Getting more confident reading and tracing through unfamiliar parts of the codebase
- Writing more consistent and predictable CSS across components

## Next week's focus

- Investigating and implementing a back button for the Entities view
- Build pop up dialog box where users choose what to export
