# Observable UX Heuristics

Use these lenses only for routes and user tasks inside the agreed audit scope.
Ask observable questions, collect evidence, and report bounded findings. Do not
redesign the product during an audit.

## Review Lenses

### System Status Visibility

- After an action, is a loading, progress, success, or failure state exposed in
  the DOM or accessibility tree within the product's expected response time?
- Can the user distinguish pending work from a completed or stalled action?
- Does status remain available long enough to perceive and act on it?

### Language and Mental-Model Match

- Do labels, values, units, and action names use the vocabulary supplied by the
  product and the user task?
- Do action labels predict the observed result, including destructive or
  irreversible effects?
- Are dates, ordering, and grouping presented consistently with the task's
  stated domain rules?

### User Control, Cancellation, and Recovery

- Can a user cancel or leave a reversible operation before commitment?
- After an error or accidental action, is a retry, undo, back, or other recovery
  path visible and operable?
- Does navigation preserve or explicitly warn about unsaved work?

### Product Consistency

- Do repeated actions, navigation patterns, field labels, and state indicators
  behave the same on the audited routes?
- Where an existing product pattern is the approved reference, does the audited
  state expose the same role, name, placement, and outcome?
- Is any exception explained by an observable task difference?

### Error Prevention and Actionable Copy

- Are irreversible or high-impact actions distinguished before activation and
  confirmed when required?
- Does validation occur early enough to prevent avoidable submission loss?
- Does error copy identify what failed, which input or action is affected, and
  the next available recovery action?

### Recognition over Recall

- Are available actions, current selections, constraints, and prior context
  visible when the user needs them?
- Must the user remember an identifier, rule, or value from another route that
  could instead be shown in the current context?
- Do labels and examples expose required formats before failure?

### Keyboard and Accessibility

- Can every in-scope interactive target be reached and operated with the
  keyboard in a task-consistent order?
- Is focus visible, retained after updates, and moved intentionally for dialogs
  or validation errors?
- Do controls and landmarks expose appropriate roles and accessible names?
- Does measurable text contrast meet the applicable project or accessibility
  target?
- At browser zoom and narrow reflow, is content retained without two-dimensional
  scrolling except where intrinsically required?

### Responsive Hierarchy and Targets

- At each planned viewport, does content reflow without hiding the primary task,
  changing its meaning, or creating unintended overlap or clipping?
- Does visual and DOM order preserve the intended task hierarchy?
- Do interactive targets measure at least `44×44` CSS pixels, or have equivalent
  non-overlapping target area justified by the product standard?

### State Completeness

- Are loading, empty, error, success, disabled, and permission states present
  when the audited flow can reach them?
- Does each state explain the current condition and the next available action?
- Are unavailable actions programmatically and visually distinguishable without
  concealing why they are unavailable?

### Content Hierarchy, Density, and Scan Order

- Do heading levels, landmarks, visual emphasis, and DOM order reveal the
  primary task before secondary details?
- Can repeated records or controls be distinguished without relying only on
  position or color?
- At each viewport, do wrapping and density preserve labels, values, and action
  associations?

## Finding Contract

Every UX finding must identify:

- the affected user task and exact route, state, and viewport;
- observed DOM, accessibility, interaction, computed, or screenshot evidence;
- severity based on demonstrated task impact; and
- a bounded recommendation that states the outcome to restore without
  prescribing a product-wide redesign.

Distinguish an inspected lens with no finding from a lens omitted because it
was out of scope or could not be reached. State unknown product rules or design
standards as evidence gaps rather than replacing them with personal taste.
