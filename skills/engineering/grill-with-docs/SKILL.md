---
name: grill-with-docs
description: A relentless interview to sharpen a plan or design, which also creates docs (ADR's and glossary) as we go.
disable-model-invocation: true
---

Call the Skill tool twice, for "grilling" and "domain-modeling".

## Closing the session

When the grilling frontier is empty and the user confirms shared understanding, do not end on an open note: state how you classify the outcome, with your reasons, and close according to that classification. The user can overrule a classification; if they do, reclassify and continue.

- **Decision**: the conclusion already landed as an ADR or a `CONTEXT.md` update during the session. Point at it, tell the user they do not need `/to-spec`, and stop.
- **Small change**: one tweak small enough to finish in the current context window. Recommend `/implement`, with reasons.
- **Multi-session build**: work sized to span several context windows. Print only these manual prompts, one step at a time: run `/to-spec`; then run `/to-tickets`. Remind the user to complete both before clearing context, because `/to-spec` synthesizes the current conversation. Do not chain into either skill yourself.

The branch between `/implement` and `/to-spec` is the user's choice. Recommend with reasons, then wait for the user's explicit choice. Never enter either path on your own.
