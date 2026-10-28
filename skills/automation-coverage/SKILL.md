---
name: automation-coverage
description: >-
  Plans and writes RHDH and Backstage plugin tests with the automation-coverage
  MCP, using Playwright MCP for UI layers and Atlassian MCP for Jira. Use when
  asked to generate tests, fill coverage gaps, add unit, integration, component,
  or e2e automation, raise patch coverage, write plugin Playwright specs,
  complete an RHIDP, RHDHBUGS, RHDHPLAN, or RHDHSUPP automation ticket, or
  create or groom QE epics and cheapest-layer stories from a Feature
  (plan_qe_from_feature, groom_qe_tickets).
---

# Automation Coverage

The **automation-coverage** MCP is the planner. It does not write files and it does not drive a browser. Execute its plan. UI layers go through **Playwright MCP** (`--caps=testing`). Jira reads and writes go through **Atlassian MCP**.

Coverage ranks gaps. It is not a merge gate. Every test exists to catch `failureToCatch`. The cheapest layer that can catch that failure wins — do not copy the same assertion into a more expensive layer.

## Call the servers

Discover a namespace with `GetDynamicTools` before the first call in a session. The coverage server's namespace contains `automation-coverage` (often `user-automation-coverage`). Playwright's contains `playwright`. Atlassian's contains `atlassian`.

If `namespaceStatus` is `needsAuth`, call that namespace's `mcp_auth`, then discover again.

Invoke with `CallDynamicTool`. Pass only fields from the live schema. If the schema and this skill disagree, follow the schema.

`cwd` is the plugin workspace or repo you are planning (`.../rhdh-plugins/workspaces/<name>`, an overlay workspace, or the repo root). Pass `workspace` (for example `boost`) when sibling workspaces must stay out of the plan. Coverage is read from that `cwd` only.

If the namespace is missing, the MCP is not connected. Setup is in the skill repo README. Do not reimplement the planner locally.

## Route

| User ask | Do |
| --- | --- |
| Generate tests, fill gaps, raise coverage, add unit / integration / component / e2e | [Write the plan](#write-the-plan) |
| Jira key or URL about missing tests, coverage, or QE automation | [jira.md](references/jira.md), then write the plan |
| Create or groom a QE epic and layer stories from a Feature | [qe.md](references/qe.md). Do not write tests |
| Product bug (wrong behavior, regression, repro steps) | Stop. Use the `bug-fix` skill |
| Only a recommendation, no new files | `recommend_automation` and stop |

## Write the plan

1. If `coverage/coverage-final.json` and `coverage/lcov.info` are both missing, run the workspace coverage script when it is cheap (`yarn test --coverage --watchAll=false` or `yarn test:all`). If you cannot run it, still plan — changed lines are treated as uncovered.
2. `list_layers` and `discover_packages` with `cwd` and, when needed, `workspace`.
3. `analyze_changes` with the same `cwd`. An empty file list means the branch matches the base: call `generate_test_plan` with `mode=workspace` so it scans `src`. Use `mode=diff` (the default) when there is a branch or uncommitted diff.
4. `coverage_gaps`, then `generate_test_plan`, with the same `cwd`, `mode`, and `workspace`. That JSON is the source of truth. A clean branch that returns zero work items means the connected server is stale — reload the MCP. Do not invent work items to fill the gap.
5. Leave `skipped` files alone.

## Execute work items

For each item with `playwrightMcp: false`:

1. Open `template` in full. If it is missing, call `inventory_tests` for that `packageId` and layer, or glob a sibling `*.test.ts(x)` and copy its imports. Do not invent `@backstage/test-utils` or `@backstage/frontend-test-utils` import paths.
2. Write the test at `outputPath` so it fails when `failureToCatch` happens.
3. Run `runCommand` from the package. Fix until it passes.
4. Read the layer notes when they apply: [rtl.md](references/rtl.md) for `component`, [backend.md](references/backend.md) for `integration`.

For each item with `playwrightMcp: true`:

1. `generate_playwright_brief` with the same `cwd`, `mode`, and `workspace`.
2. Execute that brief with Playwright MCP only. Do not write the spec from the scenario text.
3. Follow [playwright.md](references/playwright.md).
4. Run the spec until it passes, then close the browser.

`generate_layer_brief` is for one layer when the user asked for only that layer (`layerId` from `list_layers`).

After new test files, run the workspace typecheck and lint when those scripts exist.

## Layers

Custom layers come from `.automation-coverage.yaml` via `list_layers`. Builtins, cheapest first:

| Change | Layer id |
| --- | --- |
| Utils, hooks, API clients, services | `unit` |
| Backend router or plugin wiring | `unit` and `integration` |
| React component | `component` |
| React page or user flow that needs the app shell | `component` and `ui` |
| Overlay artifact metadata | `smoke` |
| Live stack or real tokens | `overlay-e2e` |
| Plugin loads in the real RHDH app | `cluster-free-e2e` |
| Helm, operator, or IdP | `cluster-e2e` |

Do not add `cluster-e2e` or `overlay-e2e` unless the plan included them.
