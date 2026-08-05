# virtuo-claude-skills

Company repository of Claude skills for the Claude Desktop app. Skills package
Virtuo's workflows and MCP tool usage patterns so Claude delivers consistent,
reliable results across the team.

## Connect to Claude Desktop

1. Open Claude Desktop → Settings → **Plugins & skills**
2. Under **Plugin Marketplaces**, click **+ Add marketplace → Git URL**
3. Paste the URL for this repository
4. Claude re-clones the repo periodically — skill updates reach all users automatically

Once connected, skills appear in the **Organization** tab of the Directory.
Install individual skill bundles from there.

> **MCP prerequisite:** Skills reference specific MCP servers and tools. Each
> skill's README lists which MCP servers must be configured in your Claude
> Desktop session for that skill to work.

## Skill catalog

### `virtuo` — Universal skills (all employees)

| Skill | Slash command | MCP servers |
|-------|--------------|-------------|
| Meeting Prep | `/meeting-prep` | `Microsoft 365`, `Hubspot`, `Granola` |
| Meeting Recap | `/meeting-recap` | `Microsoft 365`, `Granola`, `Asana` |
| Email Triage | `/email-triage` | `Microsoft 365` |
| Daily Briefing | `/daily-briefing` | `Microsoft 365` |
| Document Search | `/document-search` | `Microsoft 365` |
| Asana My Tasks | `/asana-my-tasks` | `Asana` |
| Asana Task Capture | `/asana-task-capture` | `Asana` |
| Asana OKR Update | `/asana-okr-update` | `Asana` |
| Asana OKR Review | `/asana-okr-review` | `Asana` |
| Asana Project Status | `/asana-project-status` | `Asana` |
| Notion Search | `/notion-search` | `Notion` |
| Notion Draft | `/notion-draft` | `Notion` |
| Notion Summarize | `/notion-summarize` | `Notion` |
| Notion Update | `/notion-update` | `Notion` |

### `gtm` — Go-to-Market team

| Skill | Slash command | MCP servers |
|-------|--------------|-------------|
| Pipeline Review | `/pipeline-review` | `Hubspot` |
| Funnel Metrics | `/funnel-metrics` | `Hubspot` |
| Campaign Performance | `/campaign-performance` | `Hubspot` |
| GTM Digest | `/gtm-digest` | `Hubspot`, `Granola` |
| Lead Research | `/lead-research` | `Hubspot`, `Granola`, `Web search` |
| Deal Prep | `/deal-prep` | `Microsoft 365`, `Hubspot`, `Granola`, `Web search` |

## Contributing a skill

See the authoring rules in [.github/copilot-instructions.md](.github/copilot-instructions.md).

Quick summary:
1. Copy `skills/_template/` → `skills/<skill-name>/`
2. Fill in `SKILL.md` (name, description, MCP requirements, instructions)
3. Add an entry to `.claude-plugin/marketplace.json`
4. Open a PR — skills are reviewed before merging

All skills must include an `## MCP Requirements` section specifying the exact
MCP server and tool names (e.g. `Hubspot:search_crm_objects`) so Claude makes
precise tool calls rather than guessing.
