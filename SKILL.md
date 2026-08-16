---
name: visual-driven-review
description: Use when a user asks to test, audit, inspect, validate, or explore a web UI in a real browser; requests visual regression, responsive, accessibility, adversarial, smoke, or end-to-end UI checks; or asks agents to test UI flows in parallel. Do not use for unit tests, API-only tests, or implementing UI fixes unless separately requested.
license: MIT; derived in part from Browserbase skills/ui-test
compatibility: Requires a configured Playwright MCP server plus direct screenshot image content or a runtime image viewer that can open MCP-produced artifacts. Browser installation, MCP configuration, authentication state, and application startup are external prerequisites.
---

# Visual-Driven Review

Test the UI that users actually see. A screenshot is not a receipt: every
material screenshot must be deliberately inspected, and obvious defects on an
authorized page must be reported even when they were not named in the plan.

Test and report the UI; do not implement code fixes as part of this workflow.

## Reviewer Mindset

Default to assuming the page under test contains undiscovered defects. A clean
first impression is not a conclusion — it means the required systematic sweep
has not been completed yet, not that the page is clean. Emitting `VISUAL_CLEAR`
requires having actively hunted per the mandatory checks below, not merely
having glanced at the viewport without noticing anything obviously wrong.

Many real defects — orphan spacer elements, mismatched control-cluster spacing,
content wider than its container — do not visually "jump out" in a whole-page
screenshot. They only surface when specifically measured. Do not rely on
spontaneous suspicion alone to trigger the checks in "Mandatory Micro-Level
Checks" below; run them unconditionally for every material state.

## Non-Negotiable Boundaries

- Select one configured Playwright MCP server before the run and use only that
  server's fully qualified `browser_*` tools for browser operations. Default to
  `playwright-headless`. Select `playwright-extension` only when the run
  explicitly requires the current Chrome session, cookies, login, or installed
  extensions. Do not mix servers within one test flow.
- Treat MCP configuration, Node.js, browser installation, authentication state,
  and application startup as external prerequisites. Diagnose missing
  prerequisites, but never install software or edit configuration.
- Do not modify application code or write durable Playwright Test source files.
  Require a separate user request before implementing fixes.
- Store every generated run artifact under `<project-root>/.vdr-log/`. Before
  the first artifact write, resolve the working project's root and ensure its
  root `.gitignore` contains the exact standalone entry `/.vdr-log/`. Create
  `.gitignore` when absent, or append the entry without reordering or rewriting
  existing rules. This ignore entry is the only durable project-file edit
  authorized by this workflow.
- Keep navigation, actions, data mutation, accounts, and destructive operations
  inside the authorized scope. Reporting an obvious defect in a material state
  already opened for an assigned route is required evidence collection, not
  scope expansion.
- Inspect each coverage screenshot at its original captured resolution. If a
  suspicious detail is too small, capture a focused element or region image;
  never treat a thumbnail or an unviewed screenshot path as visual review.
- When the MCP screenshot response returns only a saved path, open that artifact
  with the runtime image viewer (for example, `view_image`) before continuing.
  This inspects MCP-produced evidence; it is not a second browser interface and
  must never be used to locate or interact with page elements.
- Pair functional PASS with deterministic DOM, accessibility, URL, network,
  console, or computed evidence. A screenshot proves a visual observation, not
  functional behavior.
- Never claim a complete pass when a planned step or required visual region was
  skipped, evidence is missing, or coverage is incomplete.

## Read References Selectively

- Read `references/playwright-mcp-presets.md` when selecting a server or when a
  blocking MCP precheck fails.
- Read `references/playwright-mcp-recipes.md` before browser execution. Use
  only recipes needed by the assigned steps and visual suspicions.
- Read `references/visual-observation.md` and
  `references/visual-evidence.md` for every run before opening the target.
- Read `references/ux-heuristics.md` for every run before opening the target,
  alongside `visual-observation.md` and `visual-evidence.md`. Its core
  heuristics apply by default; a request scoped narrowly to a single functional
  check does not exempt Visual Discovery coverage from them.
- Read `references/adversarial-testing.md` whenever scope contains user input,
  mutable state, destructive actions, asynchronous behavior, or state
  transitions.
- Read `references/report-template.html` only while aggregating final results.

## Blocking Precheck and Server Lock

Choose and record the server before the first browser call:

| Need | Server choice |
| --- | --- |
| Reproducible run without existing browser state | `playwright-headless` |
| Current Chrome login, cookies, or installed extension is explicitly required | `playwright-extension` |
| Parallel workers | `playwright-headless`; extension-backed shared state is not isolation |

Verify that the selected server exposes fully qualified tools with these
suffixes. Use the exact identifiers advertised by the current runtime; do not
invent a generic `mcp__playwright__*` alias when that server is not configured.

```text
browser_navigate
browser_snapshot
browser_take_screenshot
browser_evaluate
browser_console_messages
browser_network_requests
```

Also require `browser_resize` for responsive coverage and the applicable
interaction tools for planned actions. Record whether `browser_take_screenshot`
supports element or region targets; when it does, focused artifacts are required
for emitted findings. Then:

1. Resolve the working project root. In a Git project use the repository
   top-level directory; otherwise use the explicit project root supplied for
   the run. Ensure `<project-root>/.gitignore` contains `/.vdr-log/`, create the
   run directory under `<project-root>/.vdr-log/`, and record both paths. If no
   project root can be established, stop before writing artifacts and report
   the missing prerequisite instead of falling back to a legacy or temporary
   directory.
2. Navigate to the target and capture a structural snapshot to prove that a
   real document loaded at the intended origin.
3. Verify target URL, application startup instructions, and expected revision.
   Do not guess a missing server command or substitute a different target.
4. Verify authentication inputs. Prefer test-specific storage state with
   `playwright-headless`; use extension only when reusing the live Chrome
   session is an explicit requirement.
5. Lock the selected server in the run metadata. A server failure ends the flow;
   do not silently continue on the other server.
6. Verify that screenshots return viewable image content or that the runtime
   image viewer can open a saved MCP artifact. Otherwise visual coverage is
   blocked and must be skipped, not inferred from DOM evidence.
7. Before parallel work, run the isolation gate below. Configuration text is
   not runtime isolation evidence.

Route a failed precheck by cause:

- **Selected MCP server or browser launch unavailable:** stop browser testing,
  identify the missing prerequisite, and quote only the relevant guidance from
  `references/playwright-mcp-presets.md`.
- **Target unavailable:** stop and request a reachable target or known startup
  instruction; do not redirect this failure to MCP configuration.
- **Authentication missing:** stop affected steps and request safe dedicated
  credentials or test-specific storage state.
- **Isolation failure:** stop only parallel execution, preserve evidence, and
  request acceptance before sequential fallback.

## Scope the Run

Record target revision, base URL, authorized routes, material UI states, safe
actions, data boundaries, authentication, and supplied design references. Note
whether the run is diff-driven or exploratory.

Resolve responsive-testing and viewport scope before building the coverage
matrix or making the first browser call. Viewport coverage answers where to
inspect; responsive testing separately authorizes cross-viewport comparison,
breakpoint behavior, and resize-transition assertions.

1. Determine whether responsive testing is authorized. If the user explicitly
   requested or excluded it, record that decision without asking again.
   Otherwise use the runtime's structured user-input tool: `AskUserQuestion` in
   Claude Code or `RequestUserInput` (`request_user_input`) in Codex. Ask
   `Include responsive testing?` with `Yes — compare responsive behavior` and
   `No — inspect selected viewports independently` as the choices.
2. If the user already supplied exact viewport labels or dimensions, use only
   those viewports and do not ask for them again. Otherwise inspect the
   project's viewport matrix, when present, to obtain suggested dimensions; it
   is a source of options, not authorization to test every entry.
3. With the same structured input tool, ask which viewport coverage to run and
   offer `PC only`, `Mobile only`, and `Mobile + PC`; keep the tool's free-form
   `Other` path available for exact custom dimensions such as `1280x720` or
   multiple comma-separated sizes.
4. Use project-defined PC/Mobile dimensions when available. Otherwise suggest
   `1440x900` for PC and `375x812` for Mobile. If an `Other` response does not
   contain exact `width x height` values, collect them with the same structured
   input tool before continuing.
5. Responsive testing requires at least two exact viewport sizes. When it is
   authorized but the viewport selection contains only one, collect at least
   one additional size with the same tool. When responsive testing is declined,
   multiple selected viewports remain independent visual coverage; do not add
   cross-viewport assertions or claim a responsive PASS.
6. Record the responsive-testing decision plus the confirmed viewport labels
   and exact dimensions as authorized scope. Do not silently add Tablet or any
   other breakpoint. If later evidence makes another size or responsive testing
   useful, request explicit scope expansion with the same tool before capture.

If neither structured input tool nor explicit user-supplied decisions cover
both responsive testing and viewports, stop before browser execution and report
scope selection as a missing prerequisite. Never infer responsive authorization
or fall back to a full multi-viewport matrix.

Separate two contracts:

1. **Visual discovery coverage:** route × material state × viewport. Do not
   predeclare what must be wrong; the purpose is to observe rendered
   relationships without confirmation bias.
2. **Planned verification:** functional, adversarial, accessibility, console,
   network, known visual acceptance criteria, and responsive behavior only when
   explicitly authorized.

For diff-driven work, map changed UI behavior to routes and states. For
exploratory work, define route/action/data boundaries before opening coverage.
A missing design reference limits taste-based comparison, but it does not make
intrinsic defects such as collision, clipping, occlusion, broken alignment, or
unintended overflow unknowable.

## Define Material States

A material state is a stable rendered condition that a user must perceive or
act on. The initial loaded state is always material. A resulting state is also
material when it changes task-relevant content, layout, overlays, navigation,
control availability, selection, validation, loading, empty, success, warning,
or error feedback. A visible status change after an action is therefore a new
material state even when the route does not change.

List expected material states while planning, then add any unanticipated state
actually reached during execution to the visual coverage matrix. Classify a
change as non-material only when it cannot alter user interpretation or action,
such as an unchanged render or an out-of-scope transient pointer position, and
record that reason in the planned step evidence. Do not use “non-material” to
avoid reviewing a visible result.

When an authorized route contains an easily reachable boundary or error path
that was not named in the plan — an empty filter result, a validation error
from obviously invalid input, a zero/negative/very-long value in a visible
field — trigger it once within the authorized scope and treat the resulting
render as a material state requiring Visual Discovery coverage, even though it
was not pre-declared. This is required evidence collection, not scope
expansion: it stays within already-authorized routes and safe actions. Do not
wait for the plan to schedule a state that is trivially reachable within scope.

## Plan Without Planning Away Discovery

Create stable IDs for planned checks. Give each planned check one route, initial
state, action, expected observable outcome, viewport coverage, and evidence
requirement. Add adversarial and coverage-gap checks where applicable, then
deduplicate the list.

Create a separate visual coverage row for every material state and viewport.
Workers may not add routes, actions, or data mutations, but they must report
every credible visual anomaly visible in their assigned rows. Do not give them
an expected defect list; it anchors observation to known issues.

Partition planned rows and steps by dependency. Whenever two or more test
groups can run independently, always dispatch them to separate sub-agents and
start them concurrently after the isolation gate passes. Do not execute ready
independent groups in the coordinator or serialize them merely for convenience.
A group is not independent when it has an ordering dependency, shares mutable
browser or application state, or can collide on test data or artifacts.

Give each sub-agent stable step IDs, a unique group ID and artifact directory,
a tool-call budget, and the output contracts below. The coordinator owns scope,
the isolation gate, aggregation, and coverage-gap detection; sub-agents own the
assigned browser execution and evidence collection.

## Gate Parallel Execution

Use `playwright-headless` and exactly two concurrent sub-agents for the smoke
gate:

1. Assign distinct group IDs, artifact directories, screenshot paths, nonce
   values, and viewports (`375x812` and `1440x900`). Use the shared reserved
   key `visual-driven-review:isolation`.
2. Both sub-agents navigate to the same safe origin, write their nonce, and
   resize to their assigned viewport through their own selected-server tools.
3. After both mutations complete, both re-read the reserved key and viewport.
4. Both capture unique screenshots and prove the artifacts exist at only their
   assigned paths.
5. Record results, remove the reserved key, and verify cleanup. Cleanup is
   teardown, not isolation evidence.

Parallel execution is allowed only when both sub-agents retain distinct nonce
values and viewport sizes and produce distinct artifacts. Use unique backend
test data because browser isolation does not isolate application databases. If
state or artifacts cross sub-agents, stop parallel work and report the failure;
never describe sequential fallback as parallel coverage.

After the gate passes, dispatch every currently ready independent group in the
same turn, up to the runtime's concurrency limit. When a batch finishes,
dispatch the next ready independent groups without converting them to serial
coordinator work. If sub-agents are unavailable, preserve the plan and report
parallel execution as blocked; require user acceptance before any sequential
fallback.

## Visual Discovery Pass

Run this pass for the initial page and every resulting material visual state,
before planned assertions can bias the verdict:

1. Set the intended viewport, verify retained dimensions, and compare document
   `scrollHeight` with the viewport height.
2. Capture a stable viewport screenshot immediately. View it at original
   resolution. If
   MCP returns only a path, open the saved file with the runtime image viewer.
   Screenshot creation or a path in the transcript is incomplete work. If no
   image viewer can open the evidence, emit `VISUAL_SKIP`.
3. For a scrollable page, capture a `fullPage` artifact as an overview and then
   inspect overlapping viewport segments from top to bottom. Recheck sticky and
   fixed layers in the actual segments where they appear. If any below-the-fold
   segment is not reached or viewable, mark it skipped; never call it
   `not present`.
4. Sweep the whole state in a fixed order: top/navigation, viewport edges,
   primary task/content, repeated components, fixed/overlay layers, and bottom
   controls/content. Mark absent regions as `not present`, not silently omitted.
5. Inspect relationships, not only elements: collisions, clipping, unintended
   occlusion, broken alignment or grouping, overflow, truncation/wrapping,
   icon–text and baseline anomalies, hierarchy loss, state ambiguity, and
   inconsistent repeated patterns. In addition, run the unconditional checks in
   "Mandatory Micro-Level Checks" below for every control cluster present in
   this state — do not treat them as optional or dependent on having already
   noticed something suspicious here.
6. Give every suspicion a focused visual recheck. When the selected MCP supports
   element or region screenshots, save and inspect that focused artifact before
   emitting a finding. If it does not, zoom or crop the original artifact with
   the runtime image viewer and record the capability gap. If neither recheck is
   possible, emit `VISUAL_SKIP`. Then run the smallest bounded DOM/layout or
   computed-style check that can confirm or refute the visual hypothesis.
   For suspicion-driven checks, measurement follows observation. The Mandatory
   Micro-Level Checks are the explicit exception: run their measurements
   unconditionally even without prior suspicion. A globally healthy DOM does
   not cancel a visible defect.
7. Reinspect the focused image and evidence. Emit one `VISUAL_FINDING` per
   credible issue. Emit `VISUAL_CLEAR` only when all required regions were
   actually reviewed and no credible issue remains. If capture or review cannot
   be completed, emit `VISUAL_SKIP`.

Use `references/visual-observation.md` for the observation threshold and
`references/visual-evidence.md` for the coverage record. An intrinsic defect
does not require a design image. When the only question is product taste,
branding, or an unknown design rule, record the missing reference as an
evidence gap instead of inventing a finding.

## Mandatory Micro-Level Checks

Run these checks for every control cluster (a filter bar, a button group, a
table header row, a repeated card/list-item template) in every material state,
regardless of whether anything looked suspicious in the whole-page screenshot.
These defects are frequently invisible at whole-page scale and are missed by
suspicion-triggered recheck alone:

1. **Spacing consistency.** For adjacent same-type controls (inputs, buttons,
   dropdowns, header cells), compare the gaps between each pair. Flag any gap
   that is visibly inconsistent with its neighbors, and any thin element sitting
   between controls that carries no visible label or content — this is often a
   leftover spacer/divider element rather than an intentional design choice.
2. **Content-vs-container width.** For every text-bearing cell, label, or
   header, compare the text's natural width against its rendered container
   width. Flag any case where this mismatch causes character-by-character
   wrapping, vertical/single-column text stacking, or unexpected truncation —
   this is a distinct, common failure mode from generic "overflow" and must be
   checked explicitly, not assumed to be caught by broader alignment checks.
3. **Structural residue.** Explicitly look for elements that occupy visible
   width or height but carry no content, label, or apparent function — empty
   table columns, unlabeled narrow cells, leftover divider bars. Name this
   category explicitly in findings (do not fold it into generic "broken
   alignment") since it usually indicates leftover markup (an uncleaned div,
   a width not reset to zero, a placeholder column) rather than a rendering
   glitch.
4. **Icon/button edge spacing.** For icon-only buttons and controls near a
   container edge, check whether padding to the edge is consistent with
   padding used elsewhere in the same cluster.

Save a focused screenshot of the cluster (cropped or element-targeted) as
evidence for any finding from this section, per the focused-recheck
requirement in the main Visual Discovery Pass.

## Execute Planned Checks After Looking

For an initial material state, complete the Visual Discovery Pass before acting.
For each planned action:

1. Capture a structural snapshot and locate the exact target. Never use a
   screenshot as a selector.
2. Perform one action with the applicable selected-server interaction tool.
3. If the result is a material visual state, capture and inspect it through the
   Visual Discovery Pass before evaluating the planned outcome.
4. Capture the resulting structural state and assert the expected change with
   bounded DOM, URL, accessibility, network, console, or computed evidence.
5. Record exact reproduction steps while executing; do not reconstruct them
   from memory during report aggregation.

Keep every browser operation inside the selected Playwright MCP server. Do not
use shell, Playwright CLI, direct CDP, or another browser interface to act on or
inspect the page. When the selected server cannot complete a required check,
record a skip and the evidence gap.

## Cover the Viewport Matrix

Cover exactly the user-confirmed viewport matrix from `Scope the Run`. The
standard suggestions are:

| Viewport | Size |
| --- | --- |
| PC | `1440x900` |
| Mobile | `375x812` |

Project-defined or user-supplied exact dimensions override these suggestions.
Tablet and other breakpoints are included only when the user selects them.
When responsive testing is declined, treat multiple viewport rows as
independent observations and do not emit a cross-viewport responsive verdict.

Before each viewport row, resize and verify retained dimensions. Save and
inspect a baseline viewport screenshot for every tested route and viewport.
For scrollable states, also save the full-page overview and overlapping segment
screenshots. Save every material state and failure with route or step ID,
viewport, state, and optional segment in the filename under its assigned group
directory.

## Return Structured Markers

Return exactly one protocol line for each planned step:

```text
STEP_PASS|<step-id>|<deterministic-evidence>|<screenshot-path-or-none>
STEP_FAIL|<step-id>|<expected> -> <actual>|<screenshot-path>
STEP_SKIP|<step-id>|<reason>
```

Return visual protocol lines independently of planned steps:

```text
VISUAL_FINDING|<finding-id>|<route>|<state>|<viewport>|<severity>|<observation>|<screenshot-path>
VISUAL_CLEAR|<route>|<state>|<viewport>|<reviewed-regions>|<screenshot-path>
VISUAL_SKIP|<route>|<state>|<viewport>|<reason>
```

For a state with findings, emit one `VISUAL_FINDING` per issue and no
`VISUAL_CLEAR`. A finding never completes row coverage: every required region
still needs `reviewed` or `not present`. If any region or required focused
evidence remains unreviewed, emit the findings and `VISUAL_SKIP` together so the
known issue cannot hide partial coverage. A planned visual failure can require
both its `STEP_FAIL` and a `VISUAL_FINDING`; a passing function can coexist with
an incidental `VISUAL_FINDING`. Keep marker fields on one line and replace
literal `|` characters inside prose fields with spaces.

`VISUAL_CLEAR` proves only that the listed regions were reviewed with no
credible finding; it is not a functional PASS. `VISUAL_SKIP` blocks a complete
visual pass. Put measurements, focused screenshot paths, and exact reproduction
steps in the associated finding record.

Every emitted finding requires a focused visual recheck. If element/region
screenshot support was available, the finding must reference that saved and
inspected focused artifact; a viewer-only zoom is not an acceptable substitute.
Missing required focused evidence also requires `VISUAL_SKIP` for the row.

## Aggregate Reports

Write artifacts beneath:

```text
<project-root>/.vdr-log/<run-id>/
├── <group-id>/
│   ├── screenshots/
│   └── findings.md
├── report.md
└── report.html
```

Merge markers and evidence into Markdown and a self-contained HTML report.
Include environment, selected MCP server, target revision, routes, states,
viewports, planned pass/fail/skip totals, discovered finding/clear/skip totals,
findings grouped by severity, exact reproduction, expected/actual behavior,
measurements, full and focused screenshots, console/network results, coverage
gaps, and isolation outcome. Distinguish a reviewed clean region from an
omitted region.

## Handle Blockers Honestly

- **Selected MCP server unavailable or browser launch fails:** stop and provide
  the relevant external preset; do not install or switch servers mid-flow.
- **Authentication missing:** stop affected steps and request isolated storage
  state, dedicated test credentials, or explicit authorization to use the
  extension-backed session.
- **Target unavailable:** report the URL and request a known startup instruction
  or reachable target; do not start an unknown command.
- **Worker budget exhausted:** preserve partial evidence and mark remaining
  planned and visual coverage as skipped.
- **State leakage:** disable parallel mode, preserve the gate evidence, and
  request acceptance before sequential fallback.
- **Screenshot capture or visual review fails:** emit `VISUAL_SKIP`; never infer
  visual PASS from DOM measurements alone.
- **Tool response truncates:** save and inspect a focused portion. Report any
  portion that remains unverified.

## Completion Check

Before delivery, verify:

- every planned step has exactly one `STEP_PASS`, `STEP_FAIL`, or `STEP_SKIP`;
- every material state/viewport resolves to exactly one complete row status:
  `VISUAL_CLEAR` when every region was reviewed and no finding exists; one or
  more `VISUAL_FINDING` markers with every region reviewed when findings exist;
  or `VISUAL_SKIP`, optionally alongside findings, when coverage is incomplete;
- every screenshot counted as coverage was visually inspected at original
  resolution, with focused reinspection for suspicious details;
- every required region is recorded as reviewed, `not present`, or skipped with
  a reason;
- every qualifying control cluster in every material state has a recorded
  micro-sweep result covering spacing, content-vs-container width, structural
  residue, and icon/button edge spacing; a missing cluster record blocks
  `VISUAL_CLEAR`;
- every FAIL/finding has exact route, state, viewport, reproduction, observable
  impact, screenshot, and deterministic evidence when measurable;
- no missing evidence, omitted region, or sequential fallback is disguised as
  a pass, and no complete visual pass is claimed while any `VISUAL_SKIP`
  remains; and
- no application or product-code file was changed; the only permitted project
  edit is the exact `/.vdr-log/` entry in the root `.gitignore`.

Apply a final Johari check: state verified evidence (Open), missing user-owned
design or environment context (Hidden), surfaced assumptions and downstream
risks (Blind Spot), and the next validation action for unresolved behavior
(Unknown). Deliver findings and gaps, not code fixes.

## Attribution

This curated rewrite derives testing ideas from Browserbase's MIT-licensed
[`skills/ui-test`](https://github.com/browserbase/skills/tree/6afe8663693372e59e167dfa5be37932af09ae3d/skills/ui-test),
inspected at commit `6afe8663693372e59e167dfa5be37932af09ae3d`.

The Playwright MCP workflow follows Microsoft's official
[`playwright-mcp`](https://github.com/microsoft/playwright-mcp/tree/55679f5f3d4b4f3e2534ec0ce2fc5683ba2eaf3f),
inspected at commit `55679f5f3d4b4f3e2534ec0ce2fc5683ba2eaf3f`.
