# Context

## Scope boundary

The limit of the decisions a design session may settle. It has two independent axes.

### Horizontal scope boundary

The boundary between the module or repository the session owns and neighbouring modules or repositories. A session may decide how to consume an existing capability across this boundary, but not change the neighbour that owns it.

### Vertical scope boundary

The boundary between development design and management decisions. Development design treats settled management decisions as inputs and records unresolved ones as explicit assumptions rather than deciding them.

### Management decision

A decision about whether work should happen, its priority, schedule, staffing, ownership between teams, or business acceptance. It is distinct from a decision about how to design or implement the work.

### Architecture baseline

A shared statement of the current module boundaries, module responsibilities, and architectural restrictions. It records the existing architecture rather than proposing a target architecture.
