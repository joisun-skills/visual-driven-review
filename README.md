# Visual-Driven Review

An Agent Skill for browser-based UI review that treats deliberate visual
inspection as required test evidence. It combines visual discovery with
functional, responsive, accessibility, adversarial, console, and network
checks through a configured Playwright MCP server.

The skill reports findings and evidence. It does not modify application code
or install browser tooling.

## Requirements

- An Agent Skills-compatible client
- A configured Playwright MCP server
- Direct screenshot image content or a runtime image viewer that can inspect
  MCP-produced screenshot artifacts
- A reachable target application and any required test authentication

The skill defaults to an isolated `playwright-headless` server. It uses an
extension-backed Playwright server only when a review explicitly requires the
current browser's login state, cookies, or installed extensions.

## Install

Clone the repository into your Agent Skills directory:

```bash
git clone https://github.com/joisun/visual-driven-review.git \
  ~/.agents/skills/visual-driven-review
```

Or add it to a Git-managed skills collection as a submodule:

```bash
git submodule add \
  https://github.com/joisun/visual-driven-review.git \
  path/to/skills/visual-driven-review
```

When cloning a parent repository that uses the skill as a submodule:

```bash
git clone --recurse-submodules <parent-repository-url>
```

For an existing clone:

```bash
git submodule update --init --recursive
```

## Use

Ask the Agent to audit, inspect, validate, or explore a web UI in a real
browser. The workflow covers broad visual audits, responsive and regression
reviews, accessibility observations, adversarial UI states, smoke checks, and
end-to-end UI flows. When the request does not already specify viewport sizes,
the skill asks whether to review PC, Mobile, both, or custom dimensions through
Claude Code's `AskUserQuestion` or Codex's `RequestUserInput` before browser
coverage begins. It never assumes a full multi-device matrix.

Run artifacts are written beneath the reviewed project's `.vdr-log/`
directory. The skill ensures the project's root `.gitignore` contains the
standalone entry `/.vdr-log/` before creating artifacts.

See [SKILL.md](SKILL.md) for the complete workflow and
[references/](references/) for the evidence, observation, Playwright MCP, and
reporting guides.

## Safety Boundaries

- Select one Playwright MCP server for a review and do not switch mid-flow.
- Keep navigation, actions, authentication, data mutation, and destructive
  operations inside the user-authorized scope.
- Inspect every material screenshot at its captured resolution.
- Pair functional conclusions with deterministic evidence.
- Do not claim complete visual coverage when evidence is missing or skipped.
- Do not modify application code as part of the review workflow.

## License and Attribution

Released under the [MIT License](LICENSE.txt).

This curated work derives testing ideas from Browserbase's MIT-licensed
[`skills/ui-test`](https://github.com/browserbase/skills/tree/6afe8663693372e59e167dfa5be37932af09ae3d/skills/ui-test).
The Playwright MCP workflow follows Microsoft's official
[`playwright-mcp`](https://github.com/microsoft/playwright-mcp/tree/55679f5f3d4b4f3e2534ec0ce2fc5683ba2eaf3f).
