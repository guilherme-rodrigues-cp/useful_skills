# INSTALL.md — installation playbook (for Claude Code)

This file is the **installer** for the `useful_skills` package. A teammate clones the repo
and asks Claude Code to *"read INSTALL.md and install the skills"*. When that happens, follow
these steps in order. Keep the user informed and **pause for authentication** where noted.

## Step 1 — Locate the cloned repo

Figure out the absolute path of the folder that contains this `INSTALL.md` (the cloned repo).
If Claude Code was opened inside it, that's the current working directory; otherwise ask the user
where they cloned it. Call that path `$REPO` for the rest of these steps.

## Step 2 — Activate every skill in the package

Each skill is a subfolder of `$REPO` that contains a `SKILL.md`. Claude Code only loads skills from
`~/.claude/skills/<name>/SKILL.md`, so link each one in there:

```bash
mkdir -p ~/.claude/skills
for d in "$REPO"/*/; do
  [ -f "${d}SKILL.md" ] || continue
  src="${d%/}"; name=$(basename "$src")
  ln -sfn "$src" "$HOME/.claude/skills/$name"   # symlink → stays in sync with `git pull`
  echo "activated: $name"
done
```

- Default to a **symlink** so a later `git pull` updates the active skill automatically. If the user
  would rather keep a static copy (e.g. they'll delete the clone), use `cp -R "$src" ~/.claude/skills/`.
- **Don't clobber a different skill** that happens to share a name (e.g. a personal `daily-summary`).
  If `~/.claude/skills/<name>` already exists and points somewhere else, ask the user before replacing it.

## Step 3 — Install the MCP servers each skill needs

Skills declare their required MCPs in their own `SKILL.md` (Step 0). For the current package:

| Skill | Needs |
|---|---|
| `self-organization` | **Slack** (required) · **Notion** (optional — only for meetings) |

Check what's already there with `claude mcp list`, then install **only what's missing**:

- **Notion** (official hosted):
  ```bash
  claude mcp add --transport http notion https://mcp.notion.com/mcp
  ```
- **Slack** — org-specific. If the team uses the official hosted Slack MCP:
  ```bash
  claude mcp add --transport http slack https://mcp.slack.com/mcp
  ```
  (or the official plugin: `claude plugin install slack@claude-plugins-official`). If you can't tell
  which Slack MCP the org uses, **ask the user** instead of guessing.

Never re-add a server that already appears in `claude mcp list`.

## Step 4 — Authenticate (hand off to the user, then verify)

Hosted MCPs use OAuth — their tools won't work until the user signs in. For **each newly added** server:

1. Tell the user to run **`/mcp`** (or complete the browser sign-in that pops up) and authenticate it.
2. **Wait** for them to confirm they've finished.
3. Re-run `claude mcp list` and confirm each server shows **`✓ Connected`**. If one is still
   disconnected, help them retry before moving on.

Do not skip ahead while a required server (Slack) is unauthenticated — the skill would produce an
empty digest.

## Step 5 — Confirm & hand off

- Show the user what's now active: `ls -l ~/.claude/skills` and the connected servers from `claude mcp list`.
- Tell them installation is complete and how to use it: run **`/self-organization`** (or just ask
  *"give me my daily digest"*). On first run the skill asks which channels to summarize, whether to
  include Notion meetings, the time window, and whether to schedule it.

---

### Maintainer notes
- **Adding a skill** = drop a `<name>/SKILL.md` folder in the repo; the loop in Step 2 picks it up
  automatically. If it needs new MCP servers, add a row to the table in Step 3.
- Keep skills **agnostic** (no hardcoded person/channel/id) so this installer stays a no-config handoff.
