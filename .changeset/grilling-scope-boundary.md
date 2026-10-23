---
"fore": patch
---

grilling now holds a scope boundary. Before the first round the session names the one requirement or module it owns, and the frontier only admits decisions inside it. A question touching another module is sorted before asking: facts about how that module already works get looked up, the local seam decision (which existing capability to consume, what to assume at the boundary) stays in scope, and anything that would change the other module triggers an immediate yes/no question to the user about opening a separate issue in that module's tracker. A yes files the issue on the spot with a short title and a line or two of context; a no, or having no tracker, parks it on the out-of-scope list. Filed links and parked items are shown whenever the list grows and restated at close; none of them blocks the session or gets decided inside it.
