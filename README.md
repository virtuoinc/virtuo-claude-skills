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

| Plugin name | Skills included | MCP servers required |
|-------------|----------------|----------------------|
| `microsoft365` | meeting-prep, meeting-recap, email-triage, daily-briefing, document-search | `Microsoft 365`, `Hubspot`, `Granola`, `Asana` |

## Contributing a skill

See the authoring rules in [.github/copilot-instructions.md](.github/copilot-instructions.md).

Quick summary:
1. Copy `skills/_template/` → `skills/<gerund-skill-name>/`
2. Fill in `SKILL.md` (name, description, MCP requirements, instructions)
3. Add an entry to `.claude-plugin/marketplace.json`
4. Open a PR — skills are reviewed before merging

All skills must include an `## MCP Requirements` section specifying the exact
MCP server and tool names (e.g. `Jira:create_issue`) so Claude makes precise
tool calls rather than guessing.
