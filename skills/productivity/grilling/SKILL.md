---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

**Hold the scope boundary.** Before the first round, name what this session owns: the one requirement or module the design tree is allowed to grow inside. The frontier admits only decisions that live within it. When a frontier question touches something another module owns, sort it before asking:

- A fact about how that module already works is a fact like any other: go and find it, never ask the user.
- A _seam_ decision this module must make is in scope and gets asked: which existing capability of the other module it consumes, and what this design assumes at the boundary.
- A decision that would change or extend the other module (how its internals should work, what it should offer next) is over the boundary. Do not grill it here. Ask the user, before anything else, whether to **open a separate issue** for it in that module's tracker: this is a yes/no question asked as soon as the item surfaces, not folded into the grilling. If they say yes and the session runs in a repo with an issue tracker, file it immediately with a short title and one or two lines of context (it can be triaged later), and link the issue on a separate **out of scope** list. If they say no, or there is no tracker, park it on that list as one line anyway. Either way it is a candidate for its own grilling session with that module's owners and is never decided here. This design records the assumption it takes at the boundary instead, and moves on.

For example, when the work is a new interface for other apps to call and an auth module already owns authentication, "how does this interface plug into the existing auth mechanism" is in scope; "how should callers authenticate, what should the auth module issue" is over the boundary: first ask whether to open an auth-module issue, then continue.

Show the out-of-scope list (filed issue links and parked items) whenever it grows, and restate it when the session closes. Items on it never block the frontier and never get decided in this session.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding. That confirmation covers the out-of-scope list too: each item on it is either an issue already filed or a follow-up the user chose to leave parked, never an open question left silently hanging here.
