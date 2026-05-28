# useful_skills

A package of **useful skills** for Claude Code, built for the team. Each skill is a folder with a
`SKILL.md` (front-matter `name` + `description` plus the step-by-step). Clone the repo, ask Claude
Code to install it, and you're set — Claude activates the skills and sets up any MCP servers they need.

## Install

1. **Clone the repo** (terminal):

   ```bash
   git clone https://github.com/guilherme-rodrigues-cp/useful_skills.git
   cd useful_skills
   ```

2. **Open Claude Code in that folder** and paste this prompt:

   ```
   Read INSTALL.md in this repo and install the skills for me: activate them in
   ~/.claude/skills, install any MCP servers they need, then walk me through the
   auth and verify everything is connected.
   ```

3. **Follow along.** Claude activates the skills and installs any missing MCP servers (e.g. Slack,
   Notion). When a server needs sign-in, it **pauses for you to authenticate** (browser OAuth, or run
   `/mcp`) and then checks that everything shows `✓ Connected`.

Then run **`/self-organization`** (or just ask *"give me my daily digest"*). On first run it asks
which channels to summarize, whether to include Notion meetings, the time window, and whether to
schedule it — nothing is hardcoded.

> Prefer to do it by hand? See [Manual install](#manual-install) below.

## Skills in the package

| Skill | What it does | MCPs |
|---|---|---|
| [`self-organization`](self-organization/SKILL.md) | Interactive, source-agnostic daily digest. On run it checks/installs the needed MCPs, identifies the current user, and asks what to include (channels, Notion, time window, scheduling). No per-person values are hardcoded. | Slack (required), Notion (optional) |

## Manual install

Claude Code only loads skills from one of these locations:

- **Global (any project):** `~/.claude/skills/<name>/SKILL.md`
- **Per project:** `<project>/.claude/skills/<name>/SKILL.md`

Activate a skill by copying or symlinking its folder into one of those places:

```bash
# symlink keeps it in sync with `git pull`
ln -sfn "$PWD/self-organization" ~/.claude/skills/self-organization
# or a static copy:
cp -R self-organization ~/.claude/skills/
```

Install the MCP servers it needs, only if they're missing (`claude mcp list` to check):

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http slack https://mcp.slack.com/mcp   # org-specific
```

Then authenticate the hosted servers with **`/mcp`**.

> This repo is the **source** of the skills (for versioning and sharing); the active copy lives in
> `~/.claude/skills/`. A symlinked install stays in sync with the repo on `git pull`.

## Sharing

The skills here are **agnostic and self-configuring** — they discover the current user and ask for the
rest (channels, Notion, window, scheduling) on each run, and install the MCPs they need if missing. So
there are no per-person values to swap before handing a skill to a teammate; they just clone, run the
installer, and go. New skills added here should follow the same agnostic pattern.

## Roadmap

- [x] Make `self-organization` agnostic & interactive (no hardcoded person/channels/ids).
- [x] One-prompt installer (`git clone` + `INSTALL.md`) that activates skills and sets up their MCPs.
- [ ] Persist per-user answers to `~/.claude/self-organization.config.json` so it doesn't re-ask.
- [ ] Package as a Claude Code plugin for true one-command install across the team.
- [ ] Add new skills to the package as they come up.
