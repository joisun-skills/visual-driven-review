# Adversarial Testing Patterns

Choose only patterns that threaten an in-scope user behavior. Define the
observable expected outcome before acting, and do not broaden the run into a
general component inventory.

## Input Boundaries

- Submit empty and whitespace-only values where input is accepted.
- Try malformed values and values immediately below, at, and above documented
  minimum or maximum boundaries.
- Use a bounded long Unicode value that includes multibyte characters and
  realistic whitespace.
- Paste input as well as typing it when paste behavior can affect validation or
  formatting.

Verify acceptance or rejection, retained or normalized value, field-level and
summary errors, focus destination, and whether invalid input caused a request.

## Interaction Pressure

- Repeat a safe action rapidly and test a deliberate double submit.
- Compare Enter submission with click submission.
- Interrupt loading only through an in-scope user control or navigation action.
- Exercise back, forward, and refresh after a material transition.

Verify request count, disabled or pending state, idempotent visible outcome,
history state, and recovery. Do not infer duplicate prevention from button
styling alone.

## State Transitions

- Move through empty, loading, success, error, and retry states.
- Verify disabled-to-enabled and enabled-to-disabled transitions.
- Compare dirty and saved states before and after navigation or refresh.
- Where safe credentials and scope permit, verify authenticated and expired
  session behavior.

Record the starting state, one transition trigger, resulting state, and any
lost or stale data.

## Destructive Actions

- Verify confirmation and cancel paths before any permitted destructive action.
- Verify undo when the product offers it.
- Check repeated delete and stale-state handling only against disposable test
  data.

Never run destructive tests against production data without separate explicit
authorization. Even with authorization, define the exact disposable record and
recovery boundary first. Cancellation must be tested before confirmation when
the environment's data safety is uncertain.

## Asynchronous Behavior

- Observe out-of-order response handling when a safe deterministic setup can
  produce it.
- Check duplicate-request prevention or idempotent results.
- Exercise timeout and retry behavior without inventing unsupported fault
  injection.
- Verify partial failure when the target already exposes a safe, repeatable way
  to reach that state.

Measure request identity, order, status, resulting DOM state, and recovery. If
the environment cannot deterministically create a condition, mark the check
SKIP instead of simulating evidence in prose.

## Navigation

- Open a deep link directly.
- Verify required query parameters survive relevant navigation.
- Check the final URL and state after redirects.
- Check whether an in-progress form loses state after back, forward, refresh,
  or an in-scope route transition.

## Evidence Protocol

Execute one action at a time so cause and effect remain attributable. For every
planned adversarial step, record:

1. the initial state and one action;
2. one observable expected outcome;
3. a Visual Discovery Pass before the planned assertion whenever the action
   creates a material visual state;
4. deterministic DOM, URL, accessibility, network, console, or computed-state
   evidence for the planned outcome;
5. exact reproduction steps;
6. exactly one `STEP_PASS`, `STEP_FAIL`, or `STEP_SKIP` marker; and
7. independent `VISUAL_FINDING`, `VISUAL_CLEAR`, or `VISUAL_SKIP` markers for
   each resulting material state using the contracts in `SKILL.md`.

Parallel mutation requires unique backend data per worker in addition to
browser isolation. Never let workers create, update, or delete the same record,
account state, idempotency key, or namespace.
