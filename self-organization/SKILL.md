---
name: self-organization
description: >-
  Interactive, source-agnostic daily digest from the user's own tools (Slack channels/DMs + optional
  Notion meetings) for any time window they choose — and optionally scheduled.
  On run it (0) makes sure the required MCP servers are installed, (1) identifies the current user,
  then asks what to include: which Slack channels, whether to add Notion meetings, the time window,
  and whether to schedule it.
  Use when asked for "daily summary", "daily digest", "what happened today/this week", "/self-organization".
  Nothing is hardcoded to a specific person — it discovers the user and asks for the rest.
---

# Self-organization — daily digest (interactive & agnostic)

Build a concise digest of what mattered recently across the user's **Slack** (channels + DMs) and,
optionally, their **Notion** meetings, for whatever **time window** they pick — and optionally
**schedule** it to be delivered automatically. The skill configures itself per user: it discovers
who is running it and asks for everything else. Do not assume any specific person, channel, or ID.

---

## Step 0 — Make sure the required MCP servers are installed

Before gathering anything, confirm the tools this skill needs are available; install what's missing.

1. **Check what's installed.** Run `claude mcp list` (Bash) and also check whether the tools are
   already exposed in this session (tool names like `mcp__*slack*` and `mcp__*notion*`).
2. **Slack (required).** If no Slack MCP is present, install the one your org uses. Common patterns:
   - Hosted/HTTP: `claude mcp add --transport http slack <SLACK_MCP_URL>`
   - stdio server with a token: `claude mcp add slack -- npx -y @modelcontextprotocol/server-slack`
     (needs `SLACK_BOT_TOKEN` / `SLACK_TEAM_ID` in the env)

   The exact Slack MCP/URL is org-specific — if you can't determine it, ask the user which Slack MCP
   to add instead of guessing.
3. **Notion (only if they want meetings — can be deferred until after question 2).** If missing:
   `claude mcp add --transport http notion https://mcp.notion.com/mcp`
4. **Authenticate.** Remote/hosted MCPs need OAuth — after adding one, ask the user to complete the
   browser sign-in (or run `/mcp`) before its tools will work. Re-check `claude mcp list` to confirm
   the server shows as connected.

If a required server can't be installed or authenticated, tell the user plainly and stop rather than
producing a half-empty digest.

## Step 1 — Identify the current user (no hardcoding)

Discover who is running the skill:
- **Slack:** get the logged-in user's `user_id` and display name (most Slack MCPs expose the current
  user — e.g., a whoami/auth tool or the search tool's documented logged-in user; otherwise resolve via
  the user's profile). Capture both.
- For DM search you don't even need the id — `to:me` already resolves to the current user. The id is
  needed only to send a scheduled DM to themselves (Step 5).
- Use the discovered name in the digest header.

## Step 2 — Ask the user what they want (in English)

Ask these **one at a time** — wait for each answer before asking the next, in this order. Do not
batch them into a single message:

1. **Which Slack channels/groups should I summarize?** Let the user **fill this in themselves** — do
   **not** pre-suggest or assume any specific channels (no defaults, no "the ones from before"). Ask
   them to type the channel names/handles; optionally offer to auto-detect the channels they belong to
   (`slack_search_channels`) only if they'd rather pick from a list. Resolve whatever they give to
   channel **IDs**.
2. **Do you want a summary of your Notion meetings?** (yes / no). If yes and Notion isn't installed or
   authenticated, do Step 0 for Notion now.
3. **What time window?** Offer: **last 24h**, **48h**, **72h**, **1 week**, or **custom** (let them
   type any range, e.g. "since Monday", "last 3 days", a date range).
4. **Do you want to schedule this?** (yes / no). If yes, go to Step 5 after producing the first digest.

> **Remember the answers.** Optionally persist the choices to `~/.claude/self-organization.config.json`
> (channels, include-notion, default window, schedule) so future runs can skip the questions unless the
> user asks to change them. Read that config at the start of Step 2 and confirm rather than re-ask.

## Step 3 — Gather (based on the answers)

**Time window.** Convert the chosen period to a Unix timestamp `OLDEST` (+ ISO date for Notion):

```bash
# HOURS = 24 | 48 | 72 | 168 (1 week) | or a custom value the user gave
HOURS=24
OLDEST=$(date -v-${HOURS}H +%s)              # macOS; Linux: date -d "${HOURS} hours ago" +%s
OLDEST_DATE=$(date -v-${HOURS}H +%Y-%m-%d)   # Linux: date -d "${HOURS} hours ago" +%Y-%m-%d
echo "oldest=$OLDEST  date=$OLDEST_DATE  (since $(date -r "$OLDEST" '+%a %d/%m %H:%M'))"
```

For a custom natural-language range, resolve it to the same `OLDEST` / `OLDEST_DATE`.

**Slack channels** (the ones picked in Q1). For each, `slack_read_channel` with `oldest=$OLDEST`,
`limit=100`, `response_format="concise"`; paginate while there are messages in the window.
- **Human discussion** → summarize normally (questions, proposals, decisions, debates).
- **Automated noise** (ETL/CI alerts, bot PR links, "+1 for…" approvals) → **condense and group** into a
  separate "🤖 Automated" subsection, e.g. "🤖 3 dbt ETL failure alerts", "👍 4 '+1 for…' approvals (…)".
- If a thread holds the outcome (a decision/answer), read it with `slack_read_thread`.

**Slack DMs / personal conversations.** `slack_search_public_and_private` with `query="to:me"`,
`channel_types="im,mpim"`, `after=$OLDEST`, `sort="timestamp"`, `sort_dir="desc"`,
`include_context=false`, `response_format="concise"` (full context blows the token budget — read the
full DM with `slack_read_channel` only for threads that clearly need a reply). Group by person;
highlight what needs the user's action or reply.

**Notion meetings** (only if Q2 = yes). `notion-query-meeting-notes` filtered by `created_time` on/after
`OLDEST_DATE` (the query already scopes to the current user's meetings). Skip placeholders (generic
`Meeting` title with `Last edited` ≈ `Created`). For each substantive note, `notion-fetch` and use the
`### Action Items` + `###` summary sections; **ignore the raw transcript** at the end. Pull out the
user's own action items.

## Step 4 — Assemble the digest (English, lean, scannable)

Omit empty sections. Suggested structure:

```
# Daily digest — <date / window spelled out>
_Covering the <chosen window>_

## ⚡ Needs your attention
- <pending actions/replies, direct questions, invites, the user's own action items> — link when useful

## 📝 Meetings (Notion)            ← only if included
- **<Meeting title>** (<date>): <what was discussed/decided in 2-4 lines>
  - _Action items:_ <who owns what; highlight the user's>

## 💬 Personal conversations
- **<Person/Group>:** <thread summary + action if any>

## 📊 <#channel>                    ← one section per chosen channel
**Discussion**
- <relevant human discussion>
**🤖 Automated**
- <condensed/grouped bot items>
```

Rules: if a source had nothing, write "nothing new" instead of omitting silently; always put the
user's required actions at the top (**⚡**); resolve `<@U...>` mentions to names; include Slack
permalinks when they help; **don't fabricate** — summarize only what's in the messages.

## Step 4b — Deliver the digest as a Slack DM (every run)

Right after assembling, **send the digest as a Slack DM to the current user — to that person only**:
- Use `slack_send_message` with the user's own `user_id` as the `channel_id` (a self-DM, e.g. `U…`).
- **Never** post it to a shared/public channel — it's a private DM to the user themselves.
- **Send it directly — do not ask for confirmation first.** Especially on the **first run**, deliver it
  straight to their DM so they can see the setup actually works. (Use `slack_send_message`, not the draft tool.)
- This happens on **every run** of the skill, not just scheduled ones; also show the digest in the chat.
- Return the message link to the user.

## Step 5 — Optional scheduling (if Q4 = yes)

Deliver the digest automatically, as a Slack DM to the user (themselves).

1. **Cadence & time.** Ask how often (default: weekdays) and at what local time (default: 08:00).
2. **Set it up with the `/schedule` routine** (remote agents on cron). Notes:
   - Remote routines need **Slack + Notion connected as connectors** at `claude.ai/customize/connectors`
     (the local MCPs above are not visible to remote runs). Warn the user and link them if missing.
   - The routine prompt must be **self-contained**: embed Steps 1–4b (identify the user, gather, assemble,
     DM) and end by **sending the digest as a Slack DM to the current user's `user_id`** (to that person only).
   - Convert the local time to a **UTC cron** (state the conversion back to the user to confirm).
3. Confirm the full config before creating, and share the routine link afterward.

---

### Design notes (for whoever extends this)
- **Agnostic by default:** no person, channel, or id is baked in — everything comes from Step 1
  (discovery) and Step 2 (the user's answers), optionally cached in the config file.
- **Elastic sources:** Slack is the core; Notion is opt-in; the same shape welcomes new sources later
  (email, calendar, GitHub) — add a gathering block + a section, and keep ⚡ at the top consolidating
  actions across all sources.
