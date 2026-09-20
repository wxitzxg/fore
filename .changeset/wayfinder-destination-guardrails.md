---
"mattpocock-skills": patch
---

wayfinder: harden the boundary between planning and delivery. Every child ticket now passes an admission test before creation ("when this ticket closes, what changes is ____"), so destination work such as retiring a pipeline or deleting a module is rejected as a map ticket and left for `/to-tickets` after `/to-spec`; a `task` ticket must name the open decision it unblocks. The `/to-spec` handoff is now explicitly a separate user invocation, with the session that empties the decision frontier stopping instead of writing the spec itself, and the execution-in-map override counts only when the user has written it into the Notes. A new Failure modes section catalogues the boundary crossings.
