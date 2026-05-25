# Claude Code — agentic system specifics

How Claude Code (the CLI / desktop / web agentic environment from Anthropic) layers on top of Claude models. This file covers the **system**, not the **model** — for model behavior, see `models/claude.md` and `matrix.md`.

Read this file when:
- The artifact is under `.claude/`, `~/.claude/`, or is a Claude Code-flavored CLAUDE.md
- The artifact relies on Claude Code-specific features (skills, subagents, slash commands, hooks, MCP servers, plan mode, auto mode, checkpointing)
- The user mentions Claude Code, Boris Cherny, plan mode, `/init`, `/clear`, `/compact`, `/rewind`, worktrees in the agentic sense

Sources merged here: official Claude Code docs (`code.claude.com/docs`), Anthropic engineering blog posts on context engineering and Agent Skills, Boris Cherny's tips (`howborisusesclaudecode.com`).

---

## CLAUDE.md — the persistent-context file

### What CLAUDE.md is

A markdown file Claude reads at the start of every Claude Code session. Holds project-specific conventions, commands, and gotchas the agent can't infer from code.

### Hierarchy

| Location | Scope |
|---|---|
| `~/.claude/CLAUDE.md` | All Claude Code sessions, globally |
| `./CLAUDE.md` (project root) | This project, shared via git |
| `./CLAUDE.local.md` | This project, personal — add to `.gitignore` |
| Parent dirs | Picked up automatically (useful for monorepos) |
| Child dirs | Loaded on-demand when working in that subtree |

Multiple files compose: a deep CLAUDE.md is read in addition to its parents, not instead of.

### Imports

```markdown
@AGENTS.md
@docs/git-instructions.md
@~/.claude/my-project-instructions.md
```

The `@path/to/file.md` syntax is a literal include. Use it to:
- Pull in `AGENTS.md` from a CLAUDE.md (cross-tool pattern)
- Split a long CLAUDE.md into focused files
- Layer personal overrides on top of shared rules

**When reviewing:** if a CLAUDE.md is over ~200 lines and rules are getting buried, propose splitting via `@import`. The imported file is invisible to readers of the main file but loaded by Claude Code.

### CLAUDE.md as a living correction log (Boris's frame)

Boris Cherny's framing — confirmed by the Claude Code team's practice:

> *"Anytime we see Claude do something incorrectly we add it to the CLAUDE.md, so Claude knows not to do it next time."*
>
> *"Update your CLAUDE.md so you don't make that mistake again"* — said at the end of every correction.
>
> *"Claude is eerily good at writing rules for itself."*

This reframes CLAUDE.md from "upfront documentation" to **incident log**. Each rule should be traceable to a specific failure the team observed. Rules that aren't load-bearing (no observed failure) are candidates for deletion.

**Symptom diagnostics (Anthropic official):**

| Symptom | What's wrong |
|---|---|
| Claude keeps doing X despite a rule against it | File too long; rule got buried in noise. Prune the file or move the rule up |
| Claude asks questions whose answers are in CLAUDE.md | Rule is ambiguously worded. Rewrite directly with an example |
| Rule is followed intermittently | Abstract language ("be careful with X"). Rewrite as concrete constraint |
| Claude contradicts a rule | Hedged wording ("maybe don't..."). Use declarative imperative |

### Anthropic's official CLAUDE.md include/exclude list

| ✅ Include | ❌ Exclude |
|---|---|
| Bash commands Claude can't guess (custom scripts, non-standard test invocations) | Anything Claude can figure out by reading code |
| Code style rules differing from language defaults | Standard conventions Claude already follows |
| Testing instructions and preferred test runners | Detailed API documentation (link instead) |
| Repo etiquette (branch naming, PR conventions) | Information that changes frequently (sprint goals) |
| Architectural decisions specific to this project | File-by-file descriptions of the codebase |
| Env-var quirks, required setup steps | Long explanations or tutorials |
| Common gotchas / non-obvious behaviors | Self-evident practices ("write clean code") |
| Negative scopes ("Don't touch `legacy/` without explicit request") | Aspirational statements |

### Test for each line

> *"Would removing this line cause Claude to make mistakes?"*

If no — cut. If you can't tell — flag as Assumption (Prime directive in `SKILL.md`), don't remove silently.

### Hooks complement CLAUDE.md (don't duplicate)

Anthropic: *"Hooks run scripts automatically at specific points in Claude's workflow. Unlike CLAUDE.md instructions which are advisory, hooks are deterministic."*

If the user has a `PostToolUse` hook that runs `eslint` after every edit, the CLAUDE.md doesn't need a "remember to run eslint" line. When reviewing a CLAUDE.md cluttered with rules already enforced by hooks, propose deleting them with a one-line note pointing at the hook.

---

## SKILL.md — Claude Code skills

### Frontmatter

Required: `name`, `description`. Optional: `disable-model-invocation: true` (when set, the skill only fires on explicit `/skill-name`, never on auto-invocation — use for skills with side effects).

### Description — the trigger

Anthropic explicitly: *"Pay special attention to the `name` and `description` of your skill. Claude will use these when deciding whether to trigger the skill."*

The description is the ONLY thing in trigger context. If it's vague, the skill doesn't fire.

**Recipe:**
1. Concrete verb phrase for what the skill does
2. "When to use it" trigger contexts (file names, extensions, user phrasings)
3. Slight pushiness when undertriggering ("Use proactively", "Trigger especially when...")
4. Negative boundary if a sibling skill exists ("Do NOT use when X — prefer the Y skill")

**Bilingual triggers:** if the user works in a non-English language, include English keywords too — Claude Code's heuristics match English more reliably.

### Body — progressive disclosure

Anthropic's three-level disclosure model:

1. **Level 1**: name + description (always in system prompt)
2. **Level 2**: full SKILL.md body (loaded when Claude decides the skill is relevant)
3. **Level 3+**: additional referenced files Claude reads as needed

> *"Agents with a filesystem and code execution tools don't need to read the entirety of a skill into their context window when working on a particular task."*

When SKILL.md grows past ~500 lines or starts mixing unrelated workflows, **split**:
- Keep SKILL.md as the hub (when-to-use, workflow outline, output format, pointers)
- Move detail into `references/<topic>.md` files
- Hub instructs Claude to read the relevant reference file when the situation matches

> *"If certain contexts are mutually exclusive or rarely used together, keeping the paths separate will reduce token usage."*

**The "Gotchas section" pattern** (from Thariq, Anthropic team): the highest-signal content in a skill is usually the gotchas — the things that are easy to get wrong. Surface them early and explicitly.

### Body length

Aim under ~500 lines. A bloated SKILL.md pays the full cost on every trigger. Move detail to reference files and script helpers.

### `disable-model-invocation: true`

For skills with side effects (commits, deploys, sending messages), set this so the skill ONLY fires when the user types `/skill-name`. Without it, the skill might auto-fire on a description-keyword match.

When reviewing a side-effect skill that doesn't have this flag → `[CRITICAL]` finding.

---

## Subagents (`.claude/agents/`, `~/.claude/agents/`)

### Frontmatter

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: opus
---
```

| Field | Effect |
|---|---|
| `name` | The slug used to address the subagent ("use the security-reviewer to...") |
| `description` | Delegation trigger — same rules as SKILL.md description |
| `tools` | Tool whitelist for this subagent's context |
| `model` | `opus` / `sonnet` / `haiku` — pin model. **The body's wording must be tuned for this specific model** — see `models/claude.md` |

### Description rules

Same as skills — concrete verb, "use when...", "use proactively" if undertriggering, scope boundary against sibling agents. The description is the ONLY signal for delegation.

### Body — system prompt

1. **One-sentence functional role.** "You are a senior security engineer focused on X."
2. **Numbered workflow** — 5-8 steps for what to do on invocation. More than that and the model skims it.
3. **Output format section** — define the shape parent will consume. Sections, bullet tags, length targets.
4. **Priority-tagged findings** if the subagent produces a list (`[CRITICAL]` / `[WARN]` / `[SUGGESTION]`).
5. **Explicit don'ts** where they matter — "Do not modify files", "Do not call external APIs".

Body length under ~150 lines. Past that, subagent's scope is too wide — split.

### Spawning behavior on Opus 4.7

Boris and Anthropic both report: Opus 4.7 spawns subagents more conservatively than 4.6 did. If you want aggressive delegation (e.g., "spawn a subagent for each of: frontend, backend, database"), prompt for it explicitly.

### Subagents as context isolation, not just parallelization

From Anthropic's context-engineering blog:

> *"Subagents use their own isolated context windows, and only send relevant information back to the orchestrator."*

The primary purpose is **keeping the main conversation clean**, not running in parallel. A code-research subagent that reads 30 files and reports a 200-word summary saves the main context far more than a 100-tool-call read trail would.

### Subagent + worktree isolation

Some subagent files declare `isolation: worktree` in frontmatter — Claude Code automatically creates a temporary git worktree for the subagent's work. Useful for:
- Parallel migrations across many files
- Risky changes the user wants to inspect before merging
- Multi-step refactors where mid-state shouldn't be visible to other agents

When reviewing a subagent that does heavy file mutation: consider proposing `isolation: worktree`.

---

## Slash commands (`.claude/commands/*.md`)

### Body

The user types `/command-name`, the body runs. Description matters less (user invoked explicitly), body matters more.

**Rules:**
1. Open with the task in 1-2 sentences. No backstory.
2. Reference `$ARGUMENTS` in context, not bare. *"Fix the GitHub issue described in: $ARGUMENTS"* — not *"$ARGUMENTS"*.
3. Numbered workflow.
4. Output format.
5. Keep short — body loads on every invocation.

### Slash command vs skill — when to choose which

**Slash command when:**
- The user wants explicit control (clear "do this" command)
- Operation has side effects (commits, PRs, deploys) that shouldn't fire on inference
- Task is well-scoped and doesn't need context-dependent triggering

**Skill when:**
- Auto-invocation based on context is the goal
- Task applies across many phrasings and situations

**Hybrid:** a skill with `disable-model-invocation: true` is functionally a slash command (you must type `/skill-name`) but lives in skill infrastructure. Use this when you want the skill to be discoverable in `/help` but only fire explicitly.

### Verification — the highest-leverage addition

Anthropic's official Claude Code docs:

> *"Give Claude a way to verify its work — this is the single highest-leverage thing you can do."*

For any slash command with side effects, the body must include a verification step. Examples:

| Strategy | What to write |
|---|---|
| Tests | "After implementing, run `pnpm test:unit -- --changed` and don't proceed until all pass" |
| Visual diff | "Take a screenshot of the result and compare to the design at `$ARGUMENTS`. List differences and fix them" |
| Build / lint | "Run `pnpm typecheck && pnpm lint` and fix any failures before committing" |
| Browser e2e | "Use the Chrome extension to load the local dev server and verify the user flow described above" |

When reviewing a slash command that lacks verification: usually `[CRITICAL]` if the command writes code or commits, `[IMPROVE]` if it only reads.

---

## Hooks — when wording matters and when it doesn't

Hooks are configured in `.claude/settings.json`, not in CLAUDE.md or skill files. Hook *configuration* is out of scope for this skill — but two wording-relevant interactions:

### When hooks make wording redundant

If a `PostToolUse` hook runs lint after every edit, the CLAUDE.md "remember to lint" rule is dead weight. Propose deletion + one-line pointer at the hook.

### When wording must compensate for missing hooks

If the user *doesn't* have hooks (or the CI environment doesn't run them), CLAUDE.md must carry the rules. Don't assume hook coverage in cross-vendor `AGENTS.md` — Codex's hook system is separate and may not exist on the other side.

---

## Plan mode and the Explore-Plan-Implement-Commit workflow

Anthropic's official recommended workflow:

1. **Explore** (plan mode): "read /src/auth and understand how we handle sessions"
2. **Plan** (plan mode): "I want to add Google OAuth. Create a plan."
3. **Implement** (default mode): "implement the OAuth flow from your plan"
4. **Commit**: "commit with a descriptive message and open a PR"

When reviewing a CLAUDE.md or slash command for an agentic task: if it jumps straight to implementation, propose adding a plan-mode-first step. Boris: *"Pour your energy into the plan so Claude can 1-shot the implementation."*

### When plan mode is overhead

Anthropic explicit: *"For tasks where the scope is clear and the fix is small (like fixing a typo, adding a log line, or renaming a variable) ask Claude to do it directly. Planning is most useful when you're uncertain about the approach, when the change modifies multiple files, or when you're unfamiliar with the code being modified."*

If the slash command is for a small, well-scoped operation (a typo fix, a log line, a rename), don't add planning overhead.

---

## Auto mode and permission management

`auto` mode runs a separate classifier that approves routine actions and pauses on suspicious ones. From Anthropic docs:

> *"A classifier model reviews commands before they run, blocking scope escalation, unknown infrastructure, and hostile-content-driven actions while letting routine work proceed without prompts."*

Wording implication: in CLAUDE.md / slash commands meant for auto-mode use, **don't write "ask before destructive operations"** — the classifier handles that. Instead, encode reversibility expectations explicitly:

> "If you would force-push, hard-reset, or delete more than 10 files, stop and emit a one-line summary instead. Wait for explicit approval."

For non-interactive `claude -p` runs, auto mode aborts when the classifier blocks. Wording must front-load decisions; there's no human to fall back to.

---

## Checkpoints and `/rewind`

Claude Code automatically checkpoints before changes. `/rewind` (or double-Esc) opens the rewind menu — restore conversation only, code only, or both.

Boris's tip: **rewind beats correcting** for failed attempts:

> *"Correcting keeps the failed attempt in your context and pollutes the window."*

Wording implication for slash commands and skills: when the workflow includes try-and-revert moves, mention `/rewind` as an option in the body. Don't write defensive "be careful" prose — point at the lever instead.

---

## Context-management features in wording

| Feature | When wording should reference it |
|---|---|
| `/clear` | "If context is dirty (>2 corrections, unrelated topic detour), run `/clear` and restart with a precise prompt" — for skills/commands aimed at long sessions |
| `/compact <hint>` | For long agentic tasks: "After a major phase, suggest `/compact focus on X` to keep relevant context" |
| `Esc + Esc` / `/rewind` | Failed-attempt-recovery, see above |
| `claude --continue` / `claude --resume` | Multi-session work — encode in the workflow if the artifact is a long-running playbook |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Boris: 400000 tokens for the 1M-context model. This is config, not wording — surface as a non-prompt lever |

---

## Headless mode (`claude -p`)

For non-interactive invocations (CI, pre-commit hooks, scripts):

```bash
claude -p "Migrate $file from React to Vue. Return OK or FAIL." \
  --allowedTools "Edit,Bash(git commit *)"
```

### Wording implications

Same as Codex's `codex exec` (see `agentic-systems/codex.md`):
- **No interactive clarification** — front-load every decision
- **Strict output format** for downstream parsing — `json_schema` if the output is consumed by a script
- **Exit-code contract** for blocked operations rather than "ask user" wording
- **`--allowedTools` constrains** the toolset — write the prompt assuming only those are available

### Auto mode + `-p` interaction

Auto mode aborts after repeated classifier blocks in `-p` runs. Wording must explicitly say what to do on edge cases instead of relying on user fallback.

---

## MCP and tool descriptions

Tool descriptions are wording surface. Same rule as Codex: **tool-specific guidance lives inside the tool's `description` field**, not in CLAUDE.md or the system prompt. When reviewing a CLAUDE.md cluttered with "to use the slack-mcp tool, do X" rules → propose moving into the MCP tool definition.

If the user reports a tool isn't being used despite being relevant, the fix is almost always strengthening its description with:
- Concrete verb + object for what it does
- Trigger phrasing ("when to use")
- Required arguments (named, not implicit)
- Side effects (what changes after a call)
- Common error modes

---

## Verification metrics per artifact (signals the wording is working)

Anthropic and Boris both emphasize iteration. After making changes, the user should be able to measure improvement:

| Artifact | Verification metric | How to measure |
|---|---|---|
| **CLAUDE.md** | Mistake-rate dropping | Count corrections per session before/after |
| **SKILL.md description** | Trigger rate matching intent | How often it fires when relevant; how often it fires when not |
| **Subagent description** | Delegation rate matching intent | Same as skill |
| **Subagent body** | Output predictability | Does the parent consistently get the format it needs? |
| **Slash command** | Exit success rate | Does the command produce its intended output without manual fixup? |
| **Ad-hoc prompt** | First-shot success | Did the first response solve the problem, or did it need correction? |

When a finding is `[ADD]` or `[IMPROVE]`, the user should know what to watch to confirm the change helped. Mention the metric briefly in the review.

---

## Claude Code-specific anti-patterns

These show up in Claude Code artifacts specifically:

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **Kitchen-sink CLAUDE.md** | One file mixes code style + architecture + commands + sprint context | Split via `@import`; move sprint context out (it ages) |
| **Hardcoded model name** | "When using Opus 4.7, do X" | Strip; use functional role; pin via subagent `model:` field if needed |
| **Plan-mode prose in default mode** | Slash command body says "first explore, then plan, then code" but doesn't actually enter plan mode | Either invoke plan mode explicitly or remove the prose — it's confusing the model |
| **Hook-duplicating rule** | CLAUDE.md says "always run eslint after edits" + a `PostToolUse` hook does the same | Delete the rule, leave a one-line pointer at the hook |
| **Skill without `disable-model-invocation`** for side-effect ops | Side-effect skill auto-fires on description-keyword match | Add `disable-model-invocation: true` |
| **CLAUDE.md as upfront documentation** | File reads like a tutorial; rules are aspirational | Reframe as correction log — every rule traces to a specific failure |
| **Subagent body re-prescribes reasoning** | Body has "think step by step" + numbered list of mental moves | Strip; raise `effort` if shallow reasoning is the issue |
| **CLAUDE.md duplicating AGENTS.md** | Same rules appear in both | Pick one source of truth; CLAUDE.md imports `@AGENTS.md` for shared rules + adds Claude-specific delta |
