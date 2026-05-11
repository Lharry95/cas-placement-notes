# Week 3 - Monday 5th May - 9th May 2026

## What I worked on

- Continued development on the CAS IFC Parser project
- Worked on the relationships panel — CSS styling and structural layout
- Fixed default load behaviour for collapsible panels — ensuring they start collapsed instead of open
- Identified and fixed a duplicate buttons bug on the Footprints tab (All Doors, Inward Only, Outward Only, Unknown Direction)
- Investigated a freezing bug that occurs when large IFC files are loaded
- Identified a missing back button issue in the Entities view
- Redesigned the layout and formating, changing design to two columns instead of three.

## What I learned

- How to control default UI state on page load using CSS class names:
  - Added `collapsed` to the property set div class
  - Added `hidden` to the collapsible content div class
  - This ensures panels are hidden by default and only open when a user clicks them
- That when button logic is defined in both the HTML and JavaScript files, it renders twice — removing it from the HTML and keeping it only in `UI.JS` resolved the duplication

## Challenges

- Diagnosing the large file freezing bug — when a big IFC file is loaded, clicking any tab outside the 3D viewer causes the app to freeze
- The Entities view has no back button, meaning users are stuck once they navigate into a specific entity
- Balancing CSS styling work on the relationships panel alongside bug investigation

## Solutions

- Fixed the duplicate buttons issue by removing button code from the HTML and leaving it solely in `UI.JS`
- Fixed default panel state by adding `collapsed` and `hidden` class names directly to the relevant HTML elements
- Escalated the large file freezing bug to the team as it is part of someone elses tasks
- Logged the missing back button as an unresolved issue to address next week

## Key takeaways

- CSS class names on HTML elements can define starting state — JavaScript only needs to toggle from there
- Keeping UI logic in one place (JavaScript rather than HTML) avoids duplication and is easier to maintain
- Not all bugs can be solved independently — recognising when to escalate is just as valuable as fixing things yourself
- Systematic bug identification (logging what's broken, how it behaves, and what the impact is) is a useful skill in a team codebase

## Reflection

- This week gave me more confidence in reading through existing code and identifying where problems are coming from
- Fixing the collapsible panel behaviour was satisfying — it was a new technique and it worked cleanly
- I want to develop a better understanding of how navigation and state work inside panel-based UIs so I can tackle the back button issue next week
