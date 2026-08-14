# Visual Evidence Contract

Use this contract to plan, capture, review, and grade visual coverage.
Screenshots prove visual observations only; functional PASS requires separate
deterministic evidence such as DOM, accessibility, URL, network, console, or
computed layout.

## Coverage Matrix

Create one row for every authorized route, material state, and viewport. This
matrix tracks observation independently from planned `STEP_*` checks.

| Route | State | Viewport | Visual result | Reviewed regions | Observation | Verification evidence | Full/focused screenshot paths |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Stable route ID | Initial, loading, success, error, or other named state | Label and exact dimensions | `FINDING`, `FINDING + SKIP`, `CLEAR`, or `SKIP` | top/nav; edges; main; repeated; overlay/fixed; bottom, each reviewed or `not present` | Neutral visible relationship, `no credible finding`, or skip reason | Relevant measurement, refuted suspicion, evidence gap, or `not measurable` | Existing stable paths; `none — capture failed` only for `SKIP` |

`Captured` and `reviewed` are different states. A row is not reviewed until the
image was viewed at original captured resolution, the region sweep was recorded,
and suspicious details were reinspected in a focused image when necessary.

Use the project's supplied viewport matrix when one exists. Otherwise use:

| Viewport | Size |
| --- | --- |
| Desktop | `1440x900` |
| Tablet | `768x1024` |
| Mobile | `375x812` |

Save artifacts beneath:

```text
<project-root>/.vdr-log/<run-id>/<group-id>/screenshots/<route-or-step>-<viewport>-<state>[-focus-<finding-id>].png
```

Normalize route/step, viewport, state, and optional finding segments into stable
names. Never overwrite one state or focused detail with another.

## Observation Record

For each matrix row, record the observation before deterministic assertions:

```text
route: <stable route>
state: <material state>
viewport: <label and exact size>
full screenshot: <existing path>
reviewed at original resolution: yes | no
regions: top/nav=<reviewed|not present|skip>; edges=...; main=...; repeated=...; overlay/fixed=...; bottom=...
suspicions: <none or concise hypotheses>
focused rechecks: <finding-id + screenshot + outcome, or none>
visual result: FINDING | FINDING + SKIP | CLEAR | SKIP
```

Do not derive this record later from filenames. Write it while the image is in
view so that screenshot capture cannot masquerade as visual inspection.

## Observable Visual Relationships

Use the region sweep in `visual-observation.md` and test relationships including:

- collisions, clipping, unintended occlusion, and stacking conflicts;
- broken alignment, spacing rhythm, grouping, baselines, and repeated patterns;
- horizontal or component-local overflow and responsive reflow loss;
- typography wrapping/truncation that removes information or association;
- hierarchy, scan order, state distinction, focus, hover, disabled, loading, and
  error presentation when those states are in scope;
- measurable text contrast when tooling and the applicable target permit; and
- interactive bounding boxes against a `44×44` CSS-pixel minimum target.

This list guides attention; it is not permission to ignore a different obvious
defect. Conversely, design-dependent preference claims require a supplied
reference or product standard.

## Finding, Clear, and Evidence Gaps

A `VISUAL_FINDING` requires:

- an existing inspected screenshot and exact viewport;
- a concrete observed relationship and affected user task;
- a focused visual recheck, with a saved and inspected focused screenshot when
  the selected MCP supports element or region capture;
- deterministic evidence when measurable;
- severity from the table below; and
- exact reproduction steps recorded during execution.

A finding is issue evidence, not row-completion evidence. Continue the sweep
after finding it. If any required region or focused evidence remains incomplete,
record `FINDING + SKIP` and emit both marker types.

A `VISUAL_CLEAR` requires an existing inspected screenshot, a complete region
record, and no unresolved credible suspicion. It is not a functional PASS.

If screenshot capture or sufficient visual review fails, emit `VISUAL_SKIP` and
use `none — capture failed` or the exact review blocker. DOM measurements alone
cannot turn a missing visual review into `CLEAR` or `FINDING`.

When deterministic evidence contradicts a hypothesis, record the refutation and
do not emit a finding for it. When the issue is visibly clear but not measurable
with available tools, retain the screenshot observation, state `not measurable`,
and do not invent numbers.

## Severity

| Severity | Impact threshold |
| --- | --- |
| Critical | Blocks core completion or causes destructive loss |
| High | Breaks a core flow or makes required content inaccessible |
| Medium | Materially degrades a secondary flow or viewport |
| Low | Polish issue with limited task impact |

Assign severity from demonstrated user-task impact, not visual prominence.
