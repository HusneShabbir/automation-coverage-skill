# Automation Coverage skill

Cursor skill for the [automation-coverage MCP](https://github.com/HusneShabbir/automation-coverage-mcp). The MCP decides which tests to write. This skill tells the agent how to call it, when to hand UI work to Playwright MCP, and when to write Jira issues with Atlassian MCP.

## Install in Cursor

From a checkout of this repo:

```bash
npx skills add . -a cursor -g -y
```

Or link it yourself:

```bash
ln -sfn "$(pwd)/skills/automation-coverage" ~/.cursor/skills/automation-coverage
```

Cursor loads `~/.cursor/skills` at the start of a chat. Open a new chat after installing.

The skill name is `automation-coverage`. It applies when you ask to generate tests, fill coverage gaps, add unit or e2e automation, finish an RHIDP / RHDHBUGS / RHDHPLAN / RHDHSUPP automation ticket, or create QE epics from a Feature.

## MCP servers

The skill calls three servers. Add them in Cursor Settings → MCP, or in `~/.cursor/mcp.json`. Replace `<ABS_PATH_TO_MCP_CLONE>` with the directory that contains `automation-coverage-mcp/src/index.ts`.

```json
{
  "mcpServers": {
    "atlassian": {
      "url": "https://mcp.atlassian.com/v2/mcp"
    },
    "automation-coverage": {
      "command": "npx",
      "args": ["tsx", "<ABS_PATH_TO_MCP_CLONE>/src/index.ts"]
    },
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--caps=testing"]
    }
  }
}
```

Playwright needs `--caps=testing` so `browser_generate_locator` and `browser_verify_*` exist.

Atlassian auth: restart Cursor after saving the config, toggle the `atlassian` server, ask the agent to read a ticket, and choose `redhat.atlassian.net`.

Forest paths are optional. The MCP auto-detects sibling clones named `rhdh-plugins`, `community-plugins`, `rhdh`, and `rhdh-plugin-export-overlays`. To pin them, copy `config/rhdh-forest.example.yaml` from the MCP repo to `~/.config/automation-coverage/config.yaml`.

## What the agent will do

1. `generate_test_plan` on the plugin workspace (`mode=workspace` when the branch matches the base).
2. Write each non-UI work item by mirroring the named template, aimed at `failureToCatch`.
3. For UI work items, explore with Playwright MCP, then write the spec.
4. For a Feature, create or groom a `[QE]` epic and one story per cheapest layer. It does not invent cluster tickets the planner omitted.

## License

Apache-2.0
