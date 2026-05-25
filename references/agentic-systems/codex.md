# Codex CLI specifics — wording rules per artifact

This file covers wording rules specific to OpenAI Codex CLI artifacts. For the underlying GPT-5.x model behavior, read `../models/gpt.md` first — Codex layers its own runtime conventions on top of those.

Read this file when:
- The vendor identified in Step 2a of `SKILL.md` is OpenAI / Codex, OR
- The artifact is a cross-vendor `AGENTS.md` that must work in Codex too.

When reviewing a Codex artifact, also consult `../artifacts.md` for the matching vendor-agnostic section (CLAUDE.md/AGENTS.md, SKILL.md, subagent, slash command, ad-hoc) — this file only documents Codex-specific *additions* to those rules.

---

## Codex AGENTS.md — hierarchical persistent context

Codex CLI's primary persistent-context file is `AGENTS.md`. Unlike Claude's `CLAUDE.md` (single file per scope), Codex reads a **hierarchy** of `AGENTS.md` files and concatenates them.

### Discovery hierarchy (top-down)

1. **`~/.codex/AGENTS.override.md`** if it exists, else `~/.codex/AGENTS.md` — global personal rules.
2. **Repo root `AGENTS.override.md`** if it exists, else repo root `AGENTS.md` — project-wide rules.
3. **Sub-directory `AGENTS.override.md` / `AGENTS.md`** files between the repo root and the current working directory — area-specific rules.

Codex concatenates from the top of the hierarchy down. Files closer to the working directory layer on top of (and effectively override conflicting rules from) files higher up. Total combined size is capped by `project_doc_max_bytes` (default 32 KiB) — files past the cap are silently skipped.

### Wording implications

**1. Don't put global personal preferences in repo-root `AGENTS.md`.**

Personal style, preferred verbosity, "speak Russian to me" — those go in `~/.codex/AGENTS.md` or `~/.codex/AGENTS.override.md`. Repo-root `AGENTS.md` should hold project rules that any contributor's agent would want to follow.

**2. Use sub-directory `AGENTS.md` for area-specific rules, not duplicates.**

If `frontend/` has different conventions from `backend/`, write a 5-line `AGENTS.md` in each that scopes those rules. Don't restate the project-wide rules — they're already inherited from above.

**3. `AGENTS.override.md` is a layered escape hatch, not the primary file.**

Use `.override.md` when you want to:
- Pin a temporary rule for a workshop / migration / debugging session without polluting the shared `AGENTS.md`
- Override an upstream `AGENTS.md` you don't control (e.g., in a vendored monorepo subtree)

If the user's `AGENTS.override.md` is just a copy of `AGENTS.md` with one tweak, propose merging it back and deleting the override.

**4. The 32 KiB cap is a real constraint.**

Codex doesn't error when files past the cap are dropped — they just aren't loaded. If a deep `AGENTS.md` you wrote isn't being applied, check size first. Realistic budget for a healthy project: target combined hierarchy under 8 KiB; reserve the remaining 24 KiB for room to grow without surprise truncation.

**5. Frontmatter is not used.**

Codex `AGENTS.md` is plain markdown. Don't wrap it in YAML frontmatter — Codex doesn't parse it and the user just sees `---\nname: ...\n---` as the first block of the file.

### Symptoms → wording fixes (Codex-specific)

| Symptom | What's probably wrong |
|---|---|
| Rule in repo-root `AGENTS.md` is being ignored deep in the tree | A sub-directory `AGENTS.md` overrides it without realizing. Check the tree from CWD up to repo root. |
| Rule in user's global `~/.codex/AGENTS.md` is ignored | A repo-level `AGENTS.md` likely contradicts it (project beats global). |
| Long file isn't being read at all | Combined hierarchy past `project_doc_max_bytes`. Check `~/.codex/config.toml` for the override or split the file. |
| `AGENTS.override.md` doesn't take effect | The override pattern is `name.override.md`, not `name.local.md` or `name.private.md`. Confirm filename. |

### Note for the cross-vendor case

A bare `AGENTS.md` at a project root with no `.codex/` siblings and no Codex-specific sections is a **cross-vendor AGENTS.md** — read by both Claude Code and Codex. Tune for the strictest constraint set; see `../models/gpt.md` § "Cross-vendor universal prompts".

---

## Codex subagents

Codex supports subagents conceptually similar to Claude Code's. The subagent definition file is markdown with YAML frontmatter and lives under `.codex/agents/` (project) or `~/.codex/agents/` (global).

### Frontmatter fields that affect wording review

Optional fields the parent session inherits unless overridden:
- `model` — e.g., `gpt-5.5`, `gpt-5.4`, `gpt-5.3-codex`. The body's wording must be tuned for this specific model — read the matching section in `../models/gpt.md`.
- `model_reasoning_effort` — `none` / `low` / `medium` / `high` / `xhigh`. This is the **single biggest non-wording lever** affecting subagent behavior. If the user reports under-reasoning, raise this before rewriting the body.
- `description` — the delegation trigger. Same wording rules as Claude subagent descriptions in `../artifacts.md`.

### Wording differences from Claude subagents

**1. The `model_reasoning_effort` knob shifts what belongs in the body.**

A Claude subagent body might say "before answering, list edge cases, then evaluate each". On a Codex subagent with `model_reasoning_effort: high`, that prose is redundant — the model is already reasoning that deeply. Strip step prescriptions when effort is high; keep them when effort is low/none.

**2. Outcome-first description.**

Codex subagents inherit GPT-5.x's outcome-first preference. The body should say what the agent produces, not how it works. Compare:

| Claude-style body | Codex-style body |
|---|---|
| "1. Read the migration file. 2. Check for missing rollback. 3. Run lint. 4. Output a list of issues with severity tags." | "Review the migration for safety. Output: priority-tagged list of issues (CRITICAL / WARN / INFO), each with file:line and one-line rationale. Coverage matters: surface every plausible issue; a downstream filter will rank." |

**3. Tool guidance goes in tool descriptions, not the subagent body.**

If the subagent uses a custom tool, put the tool's "when to use / required inputs / common errors" inside the tool description, not the subagent's system prompt. Same family rule from `../models/gpt.md` rule #3.

**4. Body length constraint is similar to Claude (~150 lines).**

Past that, the subagent's scope is too wide; split. Codex doesn't enforce this with a separate cap — the AGENTS.md cap is for `AGENTS.md` only — but the same context-rot logic applies.

### Common Codex subagent issues

| Issue | Fix |
|---|---|
| Subagent description has no `model_reasoning_effort` and the user reports shallow reasoning | Setting fix, not wording. Recommend setting `model_reasoning_effort: medium` or `high` first. |
| Subagent body has "think step by step" | Delete; raise `model_reasoning_effort` if needed. |
| Subagent body has 5-example few-shot block for output format | Replace with `json_schema` if the parent session supports it; otherwise keep but consider whether 1–2 examples suffice. |
| Subagent description doesn't trigger | Same recipe as Claude (`../artifacts.md` § Subagent definitions): concrete verb, "use proactively / use when…", scope boundary. |

---

## Codex skills (SKILL.md in `.codex/skills/`)

Codex skills are conceptually similar to Claude skills — a `SKILL.md` with `name` + `description` frontmatter plus markdown body. They live in `.codex/skills/<skill-name>/SKILL.md` (project) or `~/.codex/skills/<skill-name>/SKILL.md` (global).

### Frontmatter

Required: `name`, `description`. Same wording rules as Claude skills (`../artifacts.md` § SKILL.md → Description).

### Body

Same body rules as Claude skills — open with what + why, "When to use" section that repeats triggers, lean numbered workflow, output format, "When NOT to use" section, references files in a `references/` folder.

### Codex-specific notes

**1. Description must work for Codex's delegation heuristic.**

Codex's skill-trigger heuristic is similar to Claude Code's but slightly different — empirically, Codex is somewhat more conservative about auto-invoking. Lean toward more concrete trigger phrasing and explicit "use whenever..." cues.

**2. Skills aren't auto-discovered in some configurations.**

If a skill isn't triggering at all, the wording fix may not help — confirm the skill is enabled in the user's `~/.codex/config.toml` or session config first. (This is a config-not-wording issue; flag it, don't try to solve with wording.)

**3. Cross-vendor skills are possible.**

A `SKILL.md` written generically (no Claude-specific or Codex-specific assumptions) can be reused across vendors with a symlink or a copy. When reviewing such a skill, apply the cross-vendor wording rules from `../models/gpt.md`.

---

## Codex slash commands (`.codex/commands/`)

Markdown body that runs when the user types `/command-name` in Codex CLI. Optional YAML frontmatter for metadata (`description`, `argument-hint`, etc.).

### Wording rules

Same as Claude slash commands (`../artifacts.md` § Slash-command prompts):
1. Open with the task in 1–2 sentences.
2. Reference `$ARGUMENTS` (Codex uses the same `$ARGUMENTS` placeholder convention as Claude Code) in context.
3. Numbered workflow.
4. Output format.
5. Keep short — body loads on every invocation.

### Codex-specific notes

**1. Slash commands are explicitly invoked — descriptions matter less for delegation.**

The user types `/foo`, so the description is just for the help menu. Spend the wording budget on the body.

**2. `argument-hint` improves UX.**

If the command takes structured arguments (a PR number, a file path), declare `argument-hint: <PR number>` in the frontmatter — Codex shows it during typeahead.

**3. For commands that mutate state (commits, PRs, deploys), include explicit confirmation wording.**

Codex's default behavior on destructive operations depends on approval mode (`auto` / `on-request` / `manual`). Don't assume the agent will pause — write the confirmation wording into the body if reversibility matters:

> "Before pushing, summarize the diff in 3 lines and pause for user approval. Only proceed after explicit go-ahead."

(This is the action-vs-suggestion-mode lever from `../techniques.md` §9, applied to a Codex slash command.)

---

## Codex hooks

Codex supports lifecycle hooks via `hooks.json` files or inline `[hooks]` tables in `~/.codex/config.toml`. Hooks are out of scope for this skill (config, not wording) — but two wording-relevant interactions:

### When hooks make wording redundant

If the user has a hook that runs lint after every edit, the `AGENTS.md` doesn't need a "remember to run lint" line. Mention this when reviewing an `AGENTS.md` that duplicates hook behavior — propose deleting the duplicated rule with a one-line note pointing at the hook.

### When wording must compensate for missing hooks

If the user *doesn't* have hooks (or their CI environment doesn't run them), `AGENTS.md` must carry the rules. Don't assume hook coverage in cross-vendor `AGENTS.md`s — Claude Code has hooks too, but they're configured separately and may not exist on the Codex side.

---

## Headless mode (`codex exec`)

Codex CLI supports non-interactive execution via `codex exec <prompt>`. The prompt passed to `exec` is the same artifact type as an ad-hoc prompt (`../artifacts.md` § Ad-hoc prompt), but with one twist:

### Wording implications for `codex exec` prompts

**1. No interactive clarification.**

The headless run can't ask "did you mean X or Y?" — it just proceeds. Wording must front-load all decisions: success criteria, scope, what to do on ambiguity ("if the file structure doesn't match expected, exit with `EXIT_AMBIGUOUS` and don't make changes").

**2. Approval mode often defaults to non-interactive.**

In CI / automation, the approval mode is typically `auto` or skipped. The "ask before destructive" wording (`../techniques.md` §15) is irrelevant — the model can't ask. Replace with an exit-code contract:

> "If a destructive operation (force-push, hard reset, deletion of more than 10 files) is required to complete the task, do not perform it. Exit with code 2 and a one-line stderr explaining what was needed. The orchestrator will decide."

**3. Output goes to stdout for downstream consumption.**

If a CI script downstream parses the output, define the format strictly — `json_schema` if supported, otherwise a tight grammar. Don't assume a human will read the result and tolerate prose.

---

## MCP integration (configured in `~/.codex/config.toml`)

Codex supports MCP servers (STDIO and Streamable HTTP). MCP server configuration is out of scope (it's config), but **MCP tool descriptions ARE wording**:

### MCP tool descriptions

The same rule from `../models/gpt.md` rule #3 applies, doubled: tool-specific guidance goes in the MCP tool's description schema, not in `AGENTS.md` or the system prompt. When reviewing an `AGENTS.md` cluttered with "to use the slack-mcp tool, do X" instructions, the wording fix is "move into the MCP tool's description".

### Symptom: model doesn't use an MCP tool that's clearly relevant

Common cause: the tool description is too vague ("posts a message"). Strengthen it with:
- What the tool does (concrete verb + object)
- When to use it (trigger phrasing)
- Required arguments (named, not implicit)
- Side effects (what changes after a call)
- Common error modes

This mirrors the subagent / skill description rules — same delegation heuristic.

---

## Codex vs Claude Code — wording-relevant deltas

| Aspect | Claude Code | Codex CLI | Wording implication |
|---|---|---|---|
| Persistent context file | `CLAUDE.md` (and `AGENTS.md` shared) | `AGENTS.md` hierarchical with `.override.md` pattern | Codex assumes inheritance; rules in sub-dir AGENTS.md need only be the *delta*, not full restatement |
| Context cap | Soft (context rot beyond ~300 lines) | Hard (`project_doc_max_bytes`, 32 KiB default) | Codex enforces brevity automatically; tune Claude side to match for cross-vendor |
| Subagent `model:` field | Opus / Sonnet / Haiku | gpt-5.x family + `model_reasoning_effort` | Codex has an additional reasoning knob — use it before rewriting body |
| Skill location | `~/.claude/skills/<name>/SKILL.md` | `~/.codex/skills/<name>/SKILL.md` | Wording rules identical; check delegation heuristic differences in description aggressiveness |
| Slash command | `~/.claude/commands/*.md` | `~/.codex/commands/*.md` | Same `$ARGUMENTS` convention; body rules identical |
| Hooks | settings.json hooks | `hooks.json` / `config.toml [hooks]` | Wording-equivalent: same "wording vs hook" trade-off applies |
| Headless | one-shot CLI invocation | `codex exec` | Codex makes this a first-class mode; tune ad-hoc prompts for non-interactive execution |
| MCP | supported | supported (STDIO + HTTP) | MCP tool descriptions are the same wording surface on both sides |
| Reasoning depth control | `effort: low/medium/high/xhigh` | `model_reasoning_effort` (same values + `none`) | Both vendors prefer the parameter over wording; `none` is Codex-specific |

When in doubt about a Codex artifact and there's no Codex-specific rule above, fall back to the Claude rule for the same artifact type — they're more often the same than different.
