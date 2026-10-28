# Jira ticket → tests

Read the ticket with Atlassian MCP, plan with automation-coverage, then execute the plan in `SKILL.md`. This file does not create QE epics — that is [qe.md](qe.md).

## Read

Accept `RHIDP-12345`, `RHDHBUGS-…`, `RHDHPLAN-…`, `RHDHSUPP-…`, or a `redhat.atlassian.net/browse/…` URL.

1. `getAccessibleAtlassianResources` once. Reuse `cloudId`.
2. `getJiraIssue` with `view=evidence`. If auth fails, ask the user to finish OAuth for `redhat.atlassian.net`. Do not invent the ticket body.
3. Keep summary, description, acceptance criteria, component, labels, and status.

| Ticket | Route |
| --- | --- |
| Product bug (repro, wrong behavior, regression) | Stop. Use `bug-fix` |
| Create or groom a QE epic from a Feature | [qe.md](qe.md) |
| Missing tests, coverage, QE automation, new e2e scenarios | Continue below |

## Map to a workspace

Jira component → `rhdh-plugins/workspaces/<workspace>`:

| Component | Workspace |
| --- | --- |
| Adoption Insights | `adoption-insights` |
| Bulk Import | `bulk-import` |
| Extensions | `extensions` |
| Homepage | `homepage` |
| Lightspeed, Intelligent Assistant | `lightspeed` (also `intelligent-assistant`) |
| Global Header | `global-header` |
| Quickstart | `quickstart` |
| Scorecard | `scorecard` |
| Orchestrator | `orchestrator` |
| MCP | `mcp-integrations` |
| Boost, AI Catalog | `boost` |
| Theme | `theme` |
| DCM | `dcm` |

`cwd` is the absolute path of that workspace inside the local `rhdh-plugins` clone. The MCP finds the clone by walking up from `cwd` or from `~/.config/automation-coverage/config.yaml`. If the component is unknown, ask.

Check out a feature branch only when the ticket or the user names one. Otherwise plan the current branch, or `mode=workspace` when the branch matches the base.

## Plan and finish

Cross-check `generate_test_plan` against the acceptance criteria. Add a work item only for a scenario the ticket requires and the plan omitted. Still use the cheapest layer, and still do not duplicate an assertion.

Execute every work item using the skill's execute steps.

Comment on the ticket with layer, file, test name, commands, and anything skipped. Transition status only when the user asks. Open a PR only when the user asks (`raise-pr` when that skill is available).
