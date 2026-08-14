# Playwright MCP Server Selection and Prerequisite Presets

These are user-facing prerequisite examples. Never install dependencies or edit
configuration while executing a UI-test run.

## Select Before the Run

| Scenario | Select | Reason |
| --- | --- | --- |
| Public target, local fixture, dedicated credentials, or test storage state | `playwright-headless` | Isolated, reproducible, and safe for parallel validation |
| Target explicitly requires the current Chrome login, cookies, or installed extension | `playwright-extension` | Reuses user browser state; not independent isolation |

Lock one server in run metadata and use only its fully qualified tools. Do not
start on one server and silently continue on the other. If authentication needs
change the selection, stop the current flow and re-plan a new run.

## Default Headless Preset

```toml
[mcp_servers.playwright-headless]
enabled = true
command = "npx"
args = [
  "-y",
  "@playwright/mcp@latest",
  "--isolated",
  "--browser",
  "chrome",
  "--headless",
]
```

An existing Node version manager can wrap the same command when already
configured; a particular Node version is not part of the Skill contract.

## Authenticated Isolated Tests

Prefer a safe, test-specific storage-state file with `playwright-headless`:

```text
--storage-state=/absolute/path/to/test-storage-state.json
```

Use dedicated test accounts and unique backend data. Browser-local isolation
does not isolate shared application databases.

## Extension Preset

Use only when reusing the current Chrome session or installed extensions is an
explicit requirement:

```toml
[mcp_servers.playwright-extension]
enabled = true
command = "npx"
args = [
  "-y",
  "@playwright/mcp@latest",
  "--extension",
]
```

Configure any extension token through the environment's secret mechanism. Do
not place credentials in reports, screenshots, Skill files, or chat output.

## Parallel Safety

Use `playwright-headless` for parallel workers. Do not treat an extension
connection, shared browser context, or shared persistent profile as isolation.
A persistent profile can serve only one browser instance. Even with
`--isolated`, run the Skill's two-worker state smoke gate because host runtimes
may still share one MCP client.

After changing external configuration, reload the agent environment before the
Skill retries its precheck.
