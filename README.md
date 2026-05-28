# useful_skills

A package of **useful skills** for the team, built as a **plugin** so it works in both **Claude Code**
and **Claude Cowork**. Each skill is a folder with a `SKILL.md`; the plugin bundles them so a teammate
installs once and gets all of them.

> **Repo = a plugin marketplace.** It contains one plugin, `useful-skills` (in `plugins/useful-skills/`),
> which holds the skills under `skills/`. Adding a skill later = drop a new `skills/<name>/SKILL.md`.

## Skills in the package

| Skill | What it does | Needs |
|---|---|---|
| [`self-organization`](plugins/useful-skills/skills/self-organization/SKILL.md) | Interactive, source-agnostic daily digest. It identifies the current user and asks what to include (channels, Notion, time window, scheduling) — nothing per-person is hardcoded. | Slack (required), Notion (optional) |

## Install

Pick the path for your tool:

### A) Claude Code — plugin (recommended)

```bash
# in Claude Code, add this repo as a marketplace and install the plugin
/plugin marketplace add guilherme-rodrigues-cp/useful_skills
/plugin install useful-skills@useful-skills
```

The skills register immediately. On first run, `self-organization` installs any missing MCP servers
(Slack required, Notion optional) and **pauses for you to authenticate** (`/mcp` / browser OAuth).

### B) Claude Code — manual / from a clone

Prefer to hack on it locally? Clone and let Claude activate it:

```bash
git clone https://github.com/guilherme-rodrigues-cp/useful_skills.git
cd useful_skills
```

Then paste this into Claude Code:

```
Read INSTALL.md in this repo and install the skills for me: activate them in
~/.claude/skills, install any MCP servers they need, then walk me through the
auth and verify everything is connected.
```

### C) Claude Cowork

Cowork loads skills only through a plugin, and external data comes through **Connectors** (not the
`claude mcp` CLI). Install the `useful-skills` plugin and enable the Slack/Notion connectors —
full steps in **[COWORK.md](COWORK.md)**.

---

Then run **`/self-organization`** (or just ask *"give me my daily digest"*). It asks which channels to
summarize, whether to include Notion meetings, the time window, and — on Claude Code — whether to
schedule it.

## Repo layout

```
useful_skills/                         ← this repo = a plugin marketplace
├── .claude-plugin/marketplace.json    ← lists the plugin
├── plugins/
│   └── useful-skills/                 ← the plugin (skills + manifest)
│       ├── .claude-plugin/plugin.json
│       ├── version.json               ← Cowork re-sync trigger
│       └── skills/
│           └── self-organization/SKILL.md
├── README.md
├── INSTALL.md                         ← Claude Code clone/manual install playbook
└── COWORK.md                          ← Cowork install + connectors
```

## Design

Skills here are **agnostic and self-configuring** — they discover the current user and ask for the rest
(channels, Notion, window, scheduling) on each run, and set up the data sources they need. There are no
per-person values to swap before handing a skill to a teammate. New skills should follow the same
agnostic pattern and live under `plugins/useful-skills/skills/`.

## Roadmap

- [x] Make `self-organization` agnostic & interactive (no hardcoded person/channels/ids).
- [x] One-prompt installer (`git clone` + `INSTALL.md`) for Claude Code.
- [x] Package as a plugin/marketplace — installable in Claude Code **and** Cowork.
- [ ] Persist per-user answers to `~/.claude/self-organization.config.json` so it doesn't re-ask.
- [ ] Bundle an `.mcp.json` for auto-registering Slack/Notion on Claude Code plugin install.
- [ ] Add new skills to the package as they come up.
