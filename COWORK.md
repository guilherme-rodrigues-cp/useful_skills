# useful_skills on Claude Cowork

Cowork doesn't load loose skill folders from `~/.claude/skills/` and has no `claude mcp` CLI. Instead,
skills arrive **inside a plugin**, and external data (Slack, Notion) comes through **Connectors**. The
`useful-skills` plugin in this repo (`plugins/useful-skills/`) is already in the right shape for that.

## What's different from Claude Code

| | Claude Code | Cowork |
|---|---|---|
| How skills are added | loose folder in `~/.claude/skills/`, or a plugin | **only via a plugin** (admin-provisioned or installed from the app) |
| External data | MCP servers (`claude mcp add`) | **Connectors** (authenticated in-app / admin-provisioned) |
| Scheduling (`/schedule`) | ✅ | ❌ not available — run the skill manually |

The skill's logic is identical; only the setup mechanics above change (see the *Platform notes* section
inside `skills/self-organization/SKILL.md`).

## Install the plugin

The plugin lives at **`plugins/useful-skills/`** — it has `.claude-plugin/plugin.json`, `version.json`,
and `skills/`, which is exactly what Cowork expects.

**Option A — install from the app (per user).** Open Cowork → Plugins, add this repo's marketplace
(`.claude-plugin/marketplace.json`) and install **useful-skills**. `installationPreference` is set to
`available`, so it's an opt-in install.

**Option B — provision for the whole org (admin).** Drop the plugin directory into the org plugins folder
so it ships to everyone:

- **macOS:** `/Library/Application Support/Claude/org-plugins/useful-skills/`
- **Windows:** `C:\Program Files\Claude\org-plugins\useful-skills\`

Copy the **contents of `plugins/useful-skills/`** into that `useful-skills/` folder (so `plugin.json`
ends up at `org-plugins/useful-skills/.claude-plugin/plugin.json`). Bump `version.json` to push updates —
Cowork re-syncs when that string changes. To force the install for everyone, set
`"installationPreference": "required"` (or `"auto_install"`) in `plugin.json`.

## Set up the Connectors (instead of `claude mcp add`)

The `self-organization` skill needs **Slack** (required) and optionally **Notion**. In Cowork these are
Connectors, not CLI-installed MCPs:

1. In Cowork, go to **Settings → Connectors** (or ask an admin to provision them via `managedMcpServers`).
2. Enable **Slack** and, if you want meeting summaries, **Notion**.
3. **Authenticate** each (OAuth in the browser). Use the connector's built-in test/connection check to
   confirm the tools are discovered.

Once connected, the skill's `slack_*` / `notion-*` tools resolve the same way they do in Claude Code.

## Run it

Invoke **`/self-organization`** (or just ask *"give me my daily digest"*). On first run it asks which
channels to summarize, whether to include Notion meetings, and the time window. Scheduling is skipped on
Cowork — run it whenever you want the digest.

> Sources for the Cowork plugin/connector model:
> [MCP, plugins, skills & hooks](https://claude.com/docs/cowork/3p/extensions) ·
> [Get started with Cowork](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork) ·
> [Custom connectors (remote MCP)](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
