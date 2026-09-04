# Agent Teams — Master Reference Guide

Source: https://code.claude.com/docs/en/agent-teams (Claude Code docs, as of v2.1.234+)

This is the internal reference for designing and running Claude Code **agent teams**
effectively. It's organized as: quick decision guide → setup → day-to-day control →
architecture/mechanics → best practices & prompt templates → troubleshooting →
known limitations. Skim the Playbook section first; use the rest as lookup.

---

## 1. Decision guide: should this be a team?

Agent teams add real coordination overhead and use **significantly more tokens**
than a single session (each teammate is a full separate context window). Only reach
for a team when parallelism is actually going to pay for itself.

**Use an agent team when:**
- Multiple independent angles on the same problem add value (research/review, competing debug hypotheses)
- Work naturally splits into ownership silos with little cross-talk (frontend/backend/tests, or one module per teammate)
- You want teammates to actively challenge / debate each other's findings

**Use a single session or subagents instead when:**
- Work is sequential or has many dependencies
- Tasks touch the same files (edit conflicts are a real risk)
- It's a routine/small task — coordination overhead will exceed the benefit

**Subagents vs. Agent teams**

| | Subagents | Agent teams |
|---|---|---|
| Context | Own context window; result returns to caller | Own context window; fully independent |
| Communication | Return result to caller (or message each other if Claude named them) | Teammates message each other directly |
| Coordination | Main agent manages all work | Self-coordination via messages + shared task list |
| Best for | Focused tasks where only the result matters | Complex work needing discussion/collaboration |
| Token cost | Lower — summarized back to main context | Higher — every teammate is a separate instance |

Rule of thumb: **subagents for "go fetch me an answer"; teams for "go work this out together."**

---

## 2. Playbook: how to run a good team

1. **Enable the feature** (see §3) — it's off by default.
2. **Pick a team size**: start with **3–5 teammates**. If you have ~15 independent
   tasks, 3 teammates is a good starting point. Three focused teammates usually
   beat five scattered ones.
3. **Size tasks right**: aim for self-contained units that produce a clear
   deliverable (a function, a test file, a review). Roughly **5–6 tasks per
   teammate** keeps everyone busy and gives the lead room to reassign if someone
   stalls. If the lead isn't splitting work finely enough, explicitly ask it to.
4. **Give every teammate real context in the spawn prompt.** Teammates load
   CLAUDE.md/MCP/skills automatically, but **not** the lead's conversation
   history. Spell out the task, relevant files, and constraints explicitly —
   don't assume shared context.
5. **Assign a distinct lens/ownership per teammate** so work doesn't overlap
   (different files, different review criteria, different hypotheses).
6. **Start with research/review tasks** if you're new to teams — no file-conflict
   risk, and it demonstrates the value of parallel exploration cleanly before you
   try parallel implementation.
7. **Avoid file conflicts** — never let two teammates own the same file.
8. **Monitor and steer.** Don't let a team run unattended for long stretches;
   check in, redirect bad approaches, and watch for the lead trying to implement
   work itself instead of waiting on teammates (see §8 templates).
9. **Shut teammates down explicitly by name** when their work is done, or just
   end the session — team directories clean up automatically.

### Prompt templates

**Kick off a team (parallel exploration):**
> Spawn N teammates to explore this from different angles: [angle 1], [angle 2],
> [angle 3]. Have them report findings and synthesize at the end.

**Parallel code review (distinct lenses):**
> Spawn three teammates to review PR #142: one on security implications, one on
> performance impact, one validating test coverage. Have them each review and
> report findings.

**Competing-hypothesis debugging (adversarial):**
> [Symptom description]. Spawn 5 teammates to investigate different hypotheses.
> Have them talk to each other to try to disprove each other's theories, like a
> scientific debate. Update the findings doc with whatever consensus emerges.

Why this works: sequential single-agent investigation anchors on the first
plausible theory it finds. Multiple independent investigators *actively trying to
disprove each other* converge on the theory that actually survives scrutiny.

**Context-rich spawn (don't under-specify):**
> Spawn a security reviewer teammate with the prompt: "Review the authentication
> module at src/auth/ for security vulnerabilities. Focus on token handling,
> session management, and input validation. The app uses JWT tokens stored in
> httpOnly cookies. Report any issues with severity ratings."

**Reusable role via subagent definition:**
> Spawn a teammate using the security-reviewer agent type to audit the auth
> module.
(Define the role once as a subagent — project/user/plugin/CLI scope — and reuse
it as both a delegated subagent and a team teammate. The definition's `tools`
allowlist and `model` are honored; its body is *appended* to the teammate's
system prompt. Note: `skills` and `mcpServers` frontmatter fields are **not**
applied when the definition runs as a teammate — teammates load skills/MCP from
project & user settings like a normal session.)

**Require a plan before implementation (risky/complex work):**
> Spawn an architect teammate to refactor the authentication module. Require plan
> approval before they make any changes.
(Teammate works in read-only plan mode, submits to the lead for approval/rejection
with feedback, revises if rejected, then implements once approved. Steer the
lead's approval judgment with explicit criteria, e.g. "only approve plans that
include test coverage" or "reject plans that modify the database schema.")

**Rein in a lead that's doing the work itself:**
> Wait for your teammates to complete their tasks before proceeding.

**Graceful shutdown:**
> Ask the researcher teammate to shut down.
(Lead sends a shutdown request; teammate can approve — exits gracefully — or
reject with an explanation.)

---

## 3. Enabling agent teams

**Off by default.** Without the flag, no team is set up at session start, no team
directories are written, and Claude will not spawn or propose teammates.

```json
// settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Can also be set as a plain shell environment variable. Settings precedence order
(highest wins): **managed settings > `--settings` payload > local settings >
project settings > user settings > shell env**. So a `"1"` in project/local
settings overrides a `"0"` in user settings, etc. See "Turning it back off" below.

**Side effect of enabling:** Claude may now name an ordinary subagent on its own
(so it can message it later), and **while agent teams are enabled, any named
subagent launches as a teammate** — not a subagent — even if you never asked for
a "team." This is the #1 cause of "why did Claude spawn a team I didn't ask for."
See §9 to disable this behavior.

**Requires an interactive session.** In non-interactive mode (`-p` flag,
including Agent SDK sessions), Claude never spawns teammates — a named subagent
just runs as an ordinary subagent even with the flag on.

### First run

Just describe the task and the roles you want, in natural language:

> I'm designing a CLI tool that helps developers track TODO comments across their
> codebase. Spawn three teammates to explore this from different angles: one on
> UX, one on technical architecture, one playing devil's advocate.

Claude populates a shared task list, spawns teammates, has them explore, and
synthesizes findings. Note: Claude sometimes uses subagents instead of forming a
team even when you expect one — subagents show up in the same agent panel, so
panel presence alone doesn't confirm a team formed. If that happens, ask again
and explicitly request an agent team.

**Panel controls (lead's terminal):**
- ↑/↓ — select a teammate
- Enter — open selected teammate's transcript / message it directly
- Esc — interrupt the selected teammate's current turn

Idle-row behavior: once *every* agent in the panel is idle, idle rows hide after
30s and reappear on the teammate's next turn (the teammate is still running and
addressable while hidden — just message it by name). More than 3 idle teammates
collapse into one `N idle agents` row; select + Enter to expand.

---

## 4. Controlling a running team

### Display modes

| Mode | Behavior | Requirements |
|---|---|---|
| `in-process` (default) | All teammates run inside the main terminal; select via panel | Works anywhere |
| `auto` | Split panes if already inside tmux, or iTerm2 with `it2` CLI; else falls back to in-process | tmux or iTerm2+`it2` |
| `tmux` | Split-pane mode; auto-detects tmux vs iTerm2 | tmux or iTerm2+`it2` |
| `iterm2` | iTerm2 native split panes explicitly | iTerm2 + [`it2` CLI](https://github.com/mkusaka/it2) |

Set persistently:
```json
// ~/.claude/settings.json
{ "teammateMode": "auto" }
```
Or per-session: `claude --teammate-mode auto` (experimental flag, not in `--help`).

tmux install: via system package manager. iTerm2: install `it2` CLI, then enable
**iTerm2 → Settings → General → Magic → Enable Python API**. `tmux -CC` inside
iTerm2 is the suggested entrypoint. tmux works best on macOS; has known
limitations on other OSes. Split-pane mode is **not supported** in VS Code's
integrated terminal, Windows Terminal, or Ghostty.

### Specifying teammates & models

> Spawn 4 teammates to refactor these modules in parallel. Use Sonnet for each
> teammate.

If you don't name a model, teammates run on the lead's current model, unless
`CLAUDE_CODE_SUBAGENT_MODEL` is set. (`teammateDefaultModel` setting was removed
in v2.1.234 — use the env var or name the model in the prompt instead.)

Model requests are checked against your org's `availableModels` allowlist; if
blocked, Claude Code substitutes a permitted model (family alias → newest
permitted version of that family on Anthropic API/Claude Platform on AWS;
otherwise falls back to the lead's model).

Teammates **inherit the lead's effort level** automatically.

### Talking to teammates directly

- In-process: ↑/↓ to select, Enter to view+message. `x` on selected teammate to
  stop it. Ctrl+T toggles the task list.
- Split-pane: click into the pane directly.

While viewing an in-process teammate, plain text and **skills** go to that
teammate — but built-in commands still run in the *lead's* session. A teammate's
model and fast mode are fixed at spawn time: `/model` and `/fast` only ever
affect the lead (typing them while viewing a teammate shows a notice, v2.1.199+).
`/effort` does apply to the viewed teammate's future turns (teammates track the
lead's effort level).

### Task list

- States: pending → in progress → completed. Tasks can depend on other tasks;
  a pending task with unresolved dependencies can't be claimed yet.
- Dependency unblocking is automatic — when a blocking task completes, dependents
  unblock without any action needed.
- **Lead assigns** explicitly, or **teammates self-claim** the next unassigned,
  unblocked task after finishing their current one.
- Claiming uses file locking to avoid race conditions between teammates.
- Only agents with the Task tools use the shared task list; others coordinate via
  messages only.

### Quality gates via hooks

- `TeammateIdle` — fires when a teammate is about to go idle. Exit code 2 →
  send feedback and keep it working.
- `TaskCreated` — fires on task creation. Exit code 2 → block creation + feedback.
- `TaskCompleted` — fires when a task is marked complete. Exit code 2 → block
  completion + feedback.

---

## 5. Architecture & mechanics

| Component | Role |
|---|---|
| **Team lead** | The main session; spawns teammates, coordinates, synthesizes |
| **Teammates** | Separate Claude Code instances, each on assigned tasks |
| **Task list** | Shared work items teammates claim/complete |
| **Mailbox** | Inter-agent messaging system |

- **Mailbox files**: `~/.claude/teams/{team-name}/inboxes/{agent-name}.json`.
  Malformed entries are dropped with an error on read; valid entries still
  deliver (pre-v2.1.207, one bad entry blocked the whole mailbox until manually
  deleted).
- A message (plain text or structured — plan approval, shutdown request) is
  reported "sent" only once the write to the recipient's mailbox file succeeds.
  Disk-full / non-writable mailbox dir → sender gets an error, nothing sent.
- **Team/task storage**, keyed by session-derived name `session-{first 8 chars
  of session ID}`:
  - Team config: `~/.claude/teams/{team-name}/config.json` — regenerated
    automatically at startup, updated as teammates join/idle/leave. **Removed
    when the session ends.** Don't hand-edit — it's overwritten on next state
    update, and there's no project-level equivalent (`.claude/teams/teams.json`
    in a project is *not* recognized).
  - Task list: `~/.claude/tasks/{team-name}/` — **persists locally**, never
    uploaded, so resumed sessions keep tasks. Retention follows
    `cleanupPeriodDays`.
  - Team config's `members` array records each member's name + agent ID; lead
    is always type `team-lead`; teammates carry whichever type the lead named
    (built-in or subagent definition), or nothing if none was named.

### Permissions

- Teammates **inherit the lead's permission settings at spawn time**
  (`--dangerously-skip-permissions` on the lead ⇒ all teammates too). You can
  change an individual teammate's mode after spawn, but **not** set per-teammate
  modes at spawn time.
- Teammate permission prompts surface in the **lead's** session — approve them
  there. Plan approval is the one exception: the lead grants those on your
  behalf without a separate prompt to you.

**Inter-agent messages are never treated as user consent.** When Agent A messages
Agent B via `SendMessage`, Claude Code tells B the message came from another
Claude session, not from the human. A teammate cannot approve a permission
prompt or supply consent on your behalf, and a denied action can't be laundered
through another teammate. In auto mode specifically, the classifier (a) treats a
relayed "approval" claim as untrusted input, not real confirmation, and (b)
reviews every inter-agent message — plain or structured (shutdown, plan
response) — before delivery; a blocked message never reaches the recipient. Same
rules apply to messages arriving from other Claude Code sessions outside the
team entirely.

### Context & communication

- Each teammate gets its own context window; loads CLAUDE.md/MCP/skills fresh at
  spawn, plus the spawn prompt — **not** the lead's conversation history.
- **Automatic delivery**: messages arrive without the lead polling.
- **Idle notifications**: on stopping, a teammate auto-notifies the lead — but
  the notification carries **no output**, only that it stopped. Teammates must
  actively message results or update the task list. (v2.1.198+: a teammate whose
  turn ends on an API error notifies the lead it *failed*, with error text,
  instead of looking like a normal finish.)
- **Shared task list** visible to Task-tool-enabled agents.
- **Direct messaging** by name; to reach everyone, send one message per
  recipient (no broadcast).
- Names are assigned by the lead at spawn — tell the lead what to call each
  teammate up front if you'll reference them by name later.

### Token usage

Scales with number of *active* teammates — each is a full separate instance.
Worthwhile for research/review/new-feature work; not for routine tasks where a
single session is more cost-effective. See `docs: agent team token costs` in the
official docs for detailed guidance.

---

## 6. Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| Teammates don't appear | Check agent panel (↑/↓, Enter). An idle row may just be hidden (30s after whole panel idles) — message the teammate by name to bring it back. Task may not have been "complex enough" for Claude to spawn a team — Claude decides. If you wanted split panes, verify `tmux` is on PATH (`which tmux`) or `it2`/Python API is set up for iTerm2. |
| Claude spawns teammates when you wanted subagents | Expected while the flag is on — any named subagent becomes a teammate. This breaks flows that wait on a subagent *result*, since teammates report only an idle notification (no output). Fix: disable the flag (§9), or explicitly ask for a team when you want one. |
| Too many permission prompts | All bubble up to the lead. Pre-approve common ops in permission settings before spawning teammates. |
| Teammates stop early / stall on errors | Select the teammate (Enter in-process / click pane in split mode) to inspect. Give it more instructions directly, or spawn a replacement. A message from the lead/another teammate wakes an in-process teammate waiting to retry a failed API call — it retries immediately instead of waiting out the full delay. |
| Lead declares "done" prematurely | Tell it to keep going / wait for teammates (see prompt template in §2). |
| Orphaned tmux sessions after Claude exits | `tmux ls` then `tmux kill-session -t <session-name>` to clean up manually. |

---

## 7. Known limitations (experimental feature)

- **No session resumption for in-process teammates** — `/resume`/`/rewind`
  don't restore them; the lead may try messaging teammates that no longer
  exist. Tell it to respawn.
- **Task status can lag** — teammates sometimes fail to mark tasks complete,
  blocking dependents. If a task looks stuck, verify manually and/or nudge the
  lead.
- **Shutdown can be slow** — teammates finish their current tool call/request
  before stopping.
- **One team per session** — can't create multiple named teams or share a team
  across sessions.
- **No nested teams** — only the lead manages the team; teammates can't spawn
  their own teammates.
- **No background subagents from in-process teammates** — a teammate's own
  subagents run in the foreground only (can't outlive the lead's process).
  `background: true` subagent definitions and `run_in_background: true`
  requests from a teammate fail or silently run in the foreground.
- **Lead is fixed for the session's lifetime** — no promoting a teammate to
  lead, no leadership transfer.
- **Permissions fixed at spawn** — all teammates start with the lead's mode;
  per-teammate mode changes only happen *after* spawn, not at spawn time.
- **Split panes need tmux or iTerm2** — unsupported in VS Code's integrated
  terminal, Windows Terminal, Ghostty (in-process mode works everywhere).

---

## 8. Related / alternatives

- **Subagents** (`docs/en/sub-agents`) — lightweight delegation within one
  session, no inter-agent coordination needed. Use when only the final result
  matters.
- **Git worktrees** (`docs/en/worktrees`) — manual parallel sessions you drive
  yourself, no automated team coordination.
- **Cross-session messaging** (`docs/en/cross-session-messaging`) — separate
  sessions passing messages without forming a team.

---

## 9. Quick reference

**Enable / disable:**
```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }   // enable
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "0" } }   // disable (user settings — can be overridden by higher-precedence project/local/managed settings)
```
No restart needed to disable — Claude Code reapplies settings-file `env` values
live and rereads the var each time a subagent would be named.

**Display mode:**
```json
{ "teammateMode": "auto" }   // ~/.claude/settings.json
```
```bash
claude --teammate-mode auto   # per-session, experimental
```

**Model for teammates:** name it in the spawn prompt, or set
`CLAUDE_CODE_SUBAGENT_MODEL`.

**Storage paths:**
- Mailbox: `~/.claude/teams/{team-name}/inboxes/{agent-name}.json`
- Team config: `~/.claude/teams/{team-name}/config.json` (ephemeral, removed at session end)
- Task list: `~/.claude/tasks/{team-name}/` (persists, follows `cleanupPeriodDays`)

**tmux debug:**
```bash
which tmux
tmux ls
tmux kill-session -t <session-name>
```

**Sizing rules of thumb:**
- 3–5 teammates for most workflows
- ~5–6 tasks per teammate
- ~15 independent tasks → start at 3 teammates
