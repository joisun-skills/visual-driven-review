# Playwright MCP Recipes

Use only bounded recipes needed by the planned step or an observed visual
suspicion. `PW.<tool>` below is shorthand for the fully qualified tool exposed
by the single Playwright MCP server locked for this run. Never call a literal
`PW` alias, invent `mcp__playwright__*`, or switch servers mid-flow.

When `PW.browser_take_screenshot` returns only a file path, open that
MCP-produced artifact with the runtime image viewer before recording an
observation. Image viewing is evidence inspection, not permission to use a
second browser interface or derive selectors from pixels.

## Core Recipes

| Purpose | Exact sequence | Required evidence |
| --- | --- | --- |
| Open a route | `PW.browser_navigate` → `PW.browser_snapshot` | Final URL and loaded page structure |
| Initial visual state | `PW.browser_resize` → verify viewport with bounded `PW.browser_evaluate` → `PW.browser_take_screenshot` → inspect image at original resolution → region sweep | Retained viewport, stable screenshot path, observation record, visual marker |
| Scrollable visual state | measure `scrollHeight` → viewport screenshot → `PW.browser_take_screenshot` with `fullPage: true` → inspect overview → move through overlapping viewport segments with bounded `PW.browser_evaluate` or `PW.browser_press_key` → screenshot and inspect every segment → restore top | Full-page overview, ordered segment paths/offsets, sticky/fixed rechecks, no unreached below-the-fold region |
| Locate a target | `PW.browser_snapshot` or `PW.browser_find` | Role, accessible name, text, or returned ref used for the action |
| Click | locate → `PW.browser_click` → visual-state recipe when material → `PW.browser_snapshot` | Visual marker for a material result plus deterministic changed-state evidence |
| Type | locate → `PW.browser_type` or `PW.browser_fill_form` → visual-state recipe when material → `PW.browser_snapshot` | Resulting field value, validation state, or other observable outcome |
| Select | locate → `PW.browser_select_option` → visual-state recipe when material → `PW.browser_snapshot` | Selected value and dependent state |
| Keyboard | locate → `PW.browser_press_key` → visual-state recipe when material → `PW.browser_snapshot` | Focus movement or action result |
| Suspicious visual detail | focused `PW.browser_take_screenshot` when supported → inspect focused artifact → bounded `PW.browser_evaluate`; otherwise viewer zoom/crop → record capability gap → bounded evaluate | Saved focused artifact when supported, neutral observation, relevant boxes/styles, confirmed or refuted hypothesis |
| Console | `PW.browser_console_messages` | Severity and exact relevant message, or explicit inspected-with-no-finding result |
| Network | `PW.browser_network_requests` | Failed request, status, and route impact, or explicit inspected-with-no-finding result |

A screenshot never supplies a selector and never proves functional behavior.
Capture and inspect a material resulting state before planned assertions can
anchor the visual verdict.

## Bounded Layout Checks

Use `PW.browser_evaluate` for one focused DOM or computed-layout question, not a
general test script.

### Viewport retention and horizontal overflow

```javascript
() => ({
  innerWidth,
  innerHeight,
  scrollHeight: document.documentElement.scrollHeight,
  clientWidth: document.documentElement.clientWidth,
  scrollWidth: document.documentElement.scrollWidth,
  hasHorizontalOverflow:
    document.documentElement.scrollWidth > document.documentElement.clientWidth,
  hasVerticalScroll:
    document.documentElement.scrollHeight > innerHeight
})
```

For a long page, use the full-page image as a map, not as the only detailed
review. Inspect viewport-height segments with roughly 20% vertical overlap so
content at segment boundaries is not lost. Use only selected-server tools to
move the page, record each `scrollY`, and restore the starting offset. A fixed
or sticky layer must be reviewed in viewport screenshots because a full-page
capture can represent it differently from the live viewport.

### Touch targets

```javascript
() => [...document.querySelectorAll('button, a, input, select, textarea')]
  .map((element) => {
    const rect = element.getBoundingClientRect();
    return {
      label:
        element.getAttribute('aria-label') ||
        element.textContent?.trim() ||
        element.tagName,
      width: Math.round(rect.width),
      height: Math.round(rect.height)
    };
  })
  .filter((item) => item.width < 44 || item.height < 44)
```

### Confirm or refute a suspected intersection

After the screenshot reveals a suspicious pair, adapt only the two selectors to
the exact elements identified from the structural snapshot:

```javascript
() => {
  const first = document.querySelector('[data-testid="suspect-first"]');
  const second = document.querySelector('[data-testid="suspect-second"]');
  if (!first || !second) return { measurable: false, reason: 'element missing' };

  const a = first.getBoundingClientRect();
  const b = second.getBoundingClientRect();
  const overlapWidth = Math.max(0, Math.min(a.right, b.right) - Math.max(a.left, b.left));
  const overlapHeight = Math.max(0, Math.min(a.bottom, b.bottom) - Math.max(a.top, b.top));

  return {
    measurable: true,
    first: { left: a.left, top: a.top, right: a.right, bottom: a.bottom },
    second: { left: b.left, top: b.top, right: b.right, bottom: b.bottom },
    overlapWidth,
    overlapHeight,
    intersects: overlapWidth > 0 && overlapHeight > 0
  };
}
```

Do not run this pairwise measurement across every element in the document. It
exists to test a visual hypothesis, not to replace observation with an O(n²)
collision scanner. Zero overlap refutes that specific hypothesis; visible
occlusion may still require checking a clipping ancestor or stacking layer.

Record the expression, returned values, route, state, viewport, full screenshot,
and focused screenshot. Measurement must follow the visual observation it tests.

## Failure Boundaries

- Unsafe general-purpose browser code is not a default escape hatch. If the
  selected MCP tools cannot perform a required check, mark it skipped and state
  the evidence gap.
- Do not substitute shell, direct CDP, a Playwright CLI, or another browser
  interface for browser actions or inspection.
- If a tool response is truncated, save a focused artifact and inspect the
  smallest relevant portion. Never guess what was omitted.
- A failed screenshot capture or unviewable image requires `VISUAL_SKIP`; DOM
  evidence cannot substitute for visual review.
