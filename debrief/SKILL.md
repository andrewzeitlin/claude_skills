---
name: debrief
description: Runs a full post-meeting workflow from Granola transcript to formatted minutes, GitHub issues, git commit, and Slack post. Use after meetings to process notes and create action items.
---

# /debrief — Meeting Debrief Skill

Runs a full post-meeting workflow: fetch transcript → draft minutes → create GitHub
issues → commit → post to Slack. Follow the steps below in order.

---

## Step 1 — Fetch meeting transcript from Granola

1. Call `find_recent_meetings` (limit 10) and present the list to the user.
   Ask: "Which meeting would you like to debrief?"
2. Call `get_transcript` for the chosen meeting. If `has_notes` is true, the transcript
   response may include notes — combine any explicit notes with the transcript as the
   raw source material. Notes carry more weight than transcript, since they were
   explicitly written by the user.
3. **Scan for direct instructions to Claude**: any utterance addressed to "Claude" or
   phrased as "let's ask Claude to…" or "Claude should…" must be extracted and
   treated as explicit requirements for the minutes. If the instruction implies an
   action, it still needs a human owner (not just Claude) — assign the most appropriate
   team member to execute it.
4. If Granola returns a token-expired error, ask the user to open the Granola app on
   Windows to refresh the token, then retry.

---

## Step 2 — Draft minutes

1. Summarise key decisions, facts, and action points from the transcript.
   - Follow **Chatham House rules** by default: attribute tasks by name, but do not
     attribute ideas or opinions to individuals unless the project's CLAUDE.md
     explicitly permits attribution.
   - Give particular emphasis to anything from meeting notes (explicitly written by
     the user).
   - Assign action points to named individuals wherever possible. Cross-reference the
     lab members table in the user-wide CLAUDE.md to identify likely owners.
   - Incorporate any direct Claude instructions from step 1 as explicit action items,
     with a named human owner.

2. Present the draft minutes to the user for review and approval before proceeding.

3. Ask: "Where would you like to store these minutes in the repository?"
   - Default suggestion: `Admin/Minutes/` if that folder exists in the project.
   - Filename format: `YYYY.MM.DD.MEETING_NAME.md` — use the meeting title or working
     group name. Ask the user to confirm or adjust if it's ambiguous.

4. Write the approved minutes as a `.md` file at the chosen path. Leave placeholders
   of the form `(#???)` next to each action item — these will be replaced with real
   issue links in step 4.

---

## Step 3 — Create GitHub issues for action items

For each named action item in the minutes:

1. **Check for an existing issue first**: search the repo's open issues for keywords
   from the task description. Present any likely matches to the user and ask:
   "Does this match an existing issue, or should I create a new one?"
2. If creating a new issue, follow the **`/github-projects` protocol**:
   - Ask for workstream, start/end dates, and blocked-by (per the mandatory workflow).
   - Assign the issue to the named owner(s) using their GitHub login from the lab
     members table in the user-wide CLAUDE.md.
   - Add the issue to the project's GitHub Project (check the project-level CLAUDE.md
     or `~/.claude/projects-registry.md` for the project ID).
3. Record each issue number and URL for use in step 4.

---

## Step 4 — Update minutes with issue links

Edit the minutes file to replace each `(#???)` placeholder with the real issue
number and a GitHub link, e.g. `([#42](https://github.com/OWNER/REPO/issues/42))`.

---

## Step 5 — Commit the minutes

1. Propose a commit message of the form:
   `Meeting minutes: <MEETING TITLE>, Date: YYYY-MM-DD`
   Auto-populate from the meeting metadata. Ask the user to confirm or modify.

2. Check the current git branch.
   - If not on `main`, ask: "You're on branch `<branch>`. Switch to `main` before
     committing, or commit here?"
   - Never force-switch without confirmation.

3. Stage and commit the minutes file only (not any unrelated changes).
   Push to the remote (`github` or `origin` — check with `git remote -v`).

---

## Step 6 — Post minutes to Slack

### Channel
Check the project-level CLAUDE.md or memory file for a preferred Slack channel.
If none is configured, ask the user which channel to post to, then cache it in the
project memory file for future debriefs.

### Formatting (Slack mrkdwn)
Convert the markdown minutes to Slack mrkdwn before posting:
- `## Heading` → `*Heading*`
- `**bold**` → `*bold*`
- Remove `---` horizontal rules (use a Slack `divider` block instead)
- Split into blocks of ≤ 2,900 characters, breaking on paragraph boundaries

### Mentions
On every *Owner:* / *Owners:* line, replace names with `<@USER_ID>` Slack mentions
using the lab members table in the user-wide CLAUDE.md (which includes Slack user
IDs). This makes the posted minutes actionable with live notifications.

### GitHub link
Always append a final block:
`*Full minutes on GitHub:* <URL|path/to/file.md>`
where the URL points to the file on the `main` branch of the repo.

### Posting
Post via the Slack MCP server if available, otherwise via `curl` to the Slack Web
API using the bot token in the user-level MCP config.

---

## Step 7 — Offer to review related documents

Ask: "Are there any other project documents you'd like to review or update in light
of this meeting? For example: the project README, an analysis plan, or open GitHub
issues."

If yes, help the user identify and update those documents.

---

## Notes and known limitations

- **Granola notes**: the `granola-mcp` package currently only exposes `get_transcript`.
  If meeting notes are not embedded in the transcript response, step 1 will be
  transcript-only until the package adds a dedicated notes endpoint.
- **Issue deduplication**: keyword matching against existing issues is imperfect.
  Always confirm with the user before creating a new issue.
- **Token expiry**: Granola auth tokens expire after ~6 hours. If the MCP call fails,
  ask the user to open the Granola app on Windows to trigger a token refresh.
- **Slack channel per project**: cache the chosen channel in the project memory file
  (`~/.claude/projects/<project>/memory/MEMORY.md`) under a `## Slack` heading so it
  is reused in future debriefs without asking again.