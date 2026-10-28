# Feature → QE tickets

Create or update Jira. Do not write tests in this flow.

Planner tools: `plan_qe_from_feature` (no QE epic yet) and `groom_qe_tickets` (epic or children already exist). The MCP returns payloads. Atlassian MCP creates and edits the issues.

1. `getAccessibleAtlassianResources` — cache `cloudId`.
2. Feature key (`RHDHPLAN-…` or `RHIDP-…`): `getJiraIssue` with `view=evidence`. Feature text is summary + description + acceptance criteria. If the user pasted the Feature, use that text and skip the fetch.
3. Search for an existing QE epic: `parent = <featureKey> AND summary ~ "QE"`. Load its children with `parent = <epicKey>`.
4. **Create** when no `[QE]` epic exists:
   - `plan_qe_from_feature` with `feature`, `featureKey`, `workspace`, and `cwd` when the workspace is on disk.
   - `createJiraIssue` for `epic.jira` (`issueType` Epic, `parent` = Feature key, description is the returned markdown).
   - For each `tickets[]` entry, `createJiraIssue` a Story whose `parent` is the **new epic key**, using that ticket's `jira` payload.
5. **Groom** when the epic or children exist:
   - `groom_qe_tickets` with the same Feature text plus `existingIssues: [{key, summary, description, issueType}]`.
   - `editJiraIssue` each `updates[]` row (`summary` and `description`). Markers in the description make a second groom a no-op. Keep them.
6. Do not add cluster or overlay stories the plan left out. Do not copy an L3 assertion into an L4 story.
7. Comment on the Feature with the created or updated keys.

`projectKey` defaults to `RHIDP`. Story summaries look like `[L1] …`, `[L3] …`, `[L4a] …`, `[L4b] …`.

Workspace hints match [jira.md](jira.md). Pass `workspace` through to the planner; do not re-decide the layer split after the tool returns.
