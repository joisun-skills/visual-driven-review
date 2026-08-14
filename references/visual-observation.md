# Visual Observation Protocol

Use this protocol before DOM assertions shape the verdict. The purpose is not
to admire a screenshot; it is to notice rendered relationships that selectors,
console logs, and happy-path checks do not express.

## Observation Before Explanation

For every material route, state, and viewport:

1. Capture the full intended viewport at a stable path.
2. View the actual image at its original captured resolution. When MCP returns
   only a saved path, open that file with the runtime image viewer (for example,
   `view_image`). A path or small transcript thumbnail is not an inspected
   screenshot. If the runtime cannot display it, the visual result is `SKIP`.
3. Describe visible relationships in neutral language before reading layout
   measurements: what touches, covers, clips, drifts, wraps, competes, or breaks
   an otherwise repeated pattern?
4. Sweep every required region and record it as reviewed or `not present`.
   On a scrollable page, use a full-page overview plus overlapping viewport
   segments; an unreached below-the-fold region is `SKIP`, not `not present`.
5. Turn each credible suspicion into one focused visual hypothesis.
6. Perform a focused visual recheck for every suspicion. When MCP supports an
   element or region screenshot, save and inspect it; every emitted finding must
   reference that artifact. Otherwise zoom/crop the original with the runtime
   image viewer, record the capability gap, then collect the smallest relevant
   DOM/layout or computed-style evidence. If no focused recheck is possible,
   the row is `SKIP`.
7. Confirm, downgrade to an evidence gap, or reject the hypothesis. Do not keep
   a visual claim merely because it was expected.

This order matters. DOM-first inspection anchors attention to known elements and
planned assertions; visual-first inspection preserves the chance to notice an
unexpected defect.

## Required Region Sweep

| Region | Look for relationships such as |
| --- | --- |
| Top and navigation | crowded navigation clusters, lost centering, clipped labels, inconsistent bars, ambiguous current location |
| Viewport edges | horizontal overflow, cropped shadows/content, off-canvas controls, safe-area or sticky-edge collisions |
| Primary task and content | obscured actions, label/value separation, broken grouping, wrapping that changes meaning, hierarchy loss |
| Repeated components | one item drifting from the shared grid, inconsistent padding/baselines, or an overlay relationship that breaks the repeated pattern |
| Fixed and overlay layers | dialogs, toasts, menus, banners, or floating actions unintentionally hiding required content |
| Bottom controls and content | fixed footers covering actions, clipped final rows, unreachable content, unexpected empty space |

The examples are lenses, not a defect checklist. Scan the rendered image for
other obvious broken relationships. Never conclude “clear” after checking only
the element named by the planned functional step.

## Intrinsic Defect vs Reference-Dependent Judgment

No design file is needed to report an intrinsic defect when the screenshot and
rendered geometry establish that elements unintentionally collide, content is
clipped or occluded, a control leaves the viewport, or one repeated item breaks
the visible system around it.

Require a supplied design, product rule, token, or approved comparison target
for claims such as “this color should be warmer,” “the spacing should be 24px,”
or “this hierarchy is not on-brand.” Record the missing reference as an evidence
gap. Do not convert personal taste into a defect.

## Suspicion Verification

Use a focused question instead of a broad page script:

- Do the two suspicious bounding boxes intersect, and by how many CSS pixels?
- Is required text outside its clipping ancestor or hidden behind a higher
  stacking layer?
- Does a repeated component differ materially from its siblings' alignment,
  size, gap, or baseline?
- Does the document or a specific region exceed the retained viewport?
- Does truncation/wrapping remove information or break the association between
  a label and its control?

Record the exact elements, measurement, route, state, and viewport. A zero
intersection refutes an overlap hypothesis; do not report it anyway. Conversely,
healthy global `scrollWidth`, console, network, or accessibility output cannot
refute a collision visible inside one component.

## Finding Threshold

Emit `VISUAL_FINDING` when all are true:

- the issue is visible in an inspected full or focused screenshot;
- the affected relationship and user impact can be stated concretely;
- route, state, and viewport are known;
- a focused recheck supports the observation, including a saved focused
  artifact whenever the configured MCP supports one; and
- deterministic evidence is included when the issue is measurable.

Emit `VISUAL_CLEAR` only after every required region is recorded and all
suspicions were refuted or resolved as evidence gaps. A finding does not complete
the row: emit findings and `VISUAL_SKIP` together when another region, focused
recheck, or required evidence could not be completed. Emit `VISUAL_SKIP` alone
when no credible finding was established and the row remains incomplete.

## Common Failure Modes

| Failure | Correction |
| --- | --- |
| Screenshot is captured only for the report | View it immediately and write the observation record before planned assertions |
| Agent inspects only planned selectors | Sweep the full state by region, then return to the planned step |
| DOM, console, and network are healthy, so UI is declared healthy | Treat those as separate evidence channels; inspect rendered spatial relationships |
| “Do not expand scope” suppresses incidental findings | Keep routes/actions/data fixed while reporting all credible anomalies already visible there |
| No design reference, so no visual judgment is attempted | Report intrinsic defects; reserve design-dependent judgments as evidence gaps |
| Every unusual choice becomes a defect | Recheck, measure when possible, and require a concrete affected relationship and user impact |
