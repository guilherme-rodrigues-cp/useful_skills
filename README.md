# useful_skills

A package of **useful skills** for the team — built here and exported to the rest of the team.
Each skill is a folder with a `SKILL.md` (front-matter `name` + `description` plus the step-by-step).

## Skills in the package

| Skill | What it does |
|---|---|
| [`self-organization`](self-organization/SKILL.md) | Interactive, source-agnostic daily digest (Slack + optional Notion). On run it checks/installs the needed MCPs, identifies the current user, and asks what to include (channels, Notion, time window, scheduling). No per-person values are hardcoded. |

## How to install a skill (per person)

Skills are only loaded by Claude Code if they live in one of these folders:

- **Global (any project):** `~/.claude/skills/<name>/SKILL.md`
- **Per project:** `<project>/.claude/skills/<name>/SKILL.md`

To activate a skill from this package, copy (or symlink) its folder into one of those places:

```bash
# example: activate self-organization globally
cp -R useful_skills/self-organization ~/.claude/skills/
# or, to keep it in sync with the package:
ln -s "$PWD/useful_skills/self-organization" ~/.claude/skills/self-organization
```

> This package is the **source** of the skills (for versioning and sharing). It is **not** loaded
> automatically — the active version lives in `~/.claude/skills/`. E.g., the `self-organization` here is
> a sibling of the `daily-summary` already running in Guilherme's global folder (that one was left untouched).

## Sharing

`self-organization` is **agnostic and self-configuring** — it discovers the current user and asks for
the rest (channels, Notion, window, scheduling) on each run, and installs the Slack/Notion MCPs if
they're missing. So there are no per-person values to swap before handing it to a teammate; they just
activate the folder and run it. (Newer skills added here should follow the same agnostic pattern.)

## Roadmap

- [x] Make `self-organization` agnostic & interactive (no hardcoded person/channels/ids).
- [ ] Persist per-user answers to `~/.claude/self-organization.config.json` so it doesn't re-ask.
- [ ] Package as a Claude Code plugin for one-command install across the team.
- [ ] Add new skills to the package as they come up.
