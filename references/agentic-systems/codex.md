# Codex CLI specifics — wording rules per artifact

This file covers wording rules specific to OpenAI Codex CLI artifacts. For the underlying GPT model behavior (GPT-6.x, GPT-5.6), read `../models/gpt.md` first — Codex layers its own runtime conventions on top of those.

Docs location: Codex documentation moved from developers.openai.com/codex to learn.chatgpt.com/docs.

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

**6. Memories are a second instruction source next to `AGENTS.md`.**

Codex CLI 0.165.0: *"Enabled memory v2 by default for fresh profiles and added support for importing external-agent memories into it."* The changelog says nothing about precedence between memories and `AGENTS.md`. When a Codex agent acts against a rule that is in `AGENTS.md`, check its memories (including ones imported from another agent) before rewriting the rule; keep standing rules in `AGENTS.md`, where a reviewer can see them. GPT-6 Astra's documented sensitivity to *"skills and other files"* (`../models/gpt.md` § GPT-6 Astra) makes stray instructions in any of these inputs more costly.

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
- `model` — e.g., `gpt-6.1-sol`, `gpt-6-luna`, `gpt-6.1-terra`. The body's wording must be tuned for this specific model — read the matching section in `../models/gpt.md`. A subagent still pinned to `gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex` or `gpt-5.2` is a finding under ChatGPT sign-in: those are retired or deprecated in Codex (GPT-5.5 since 2026-10-14, replaced by `gpt-6.1-sol`, or `gpt-6-luna` on Free / Go).
- `model_reasoning_effort` — the ladder is per model: `none` … `max` on GPT-6.1 Terra, GPT-6 Sol, GPT-6 Luna and GPT-5.6; `low` … `max` with no `none` on GPT-6 Astra and GPT-6.1 Sol (`../models/gpt.md` § GPT-6 family). This is the **single biggest non-wording lever** affecting subagent behavior. If the user reports under-reasoning, raise this before rewriting the body.
- `description` — the delegation trigger. Same wording rules as Claude subagent descriptions in `../artifacts.md`.

### Wording differences from Claude subagents

**1. The `model_reasoning_effort` knob shifts what belongs in the body.**

A Claude subagent body might say "before answering, list edge cases, then evaluate each". On a Codex subagent with `model_reasoning_effort: high`, that prose is redundant — the model is already reasoning that deeply. Strip step prescriptions when effort is high; keep them when effort is low/none.

Since Codex CLI 0.165.0 (2026-10-27) *"reasoning effort changes during a conversation"* are enabled by default for supported models, and 0.164.0 added inline `/model` arguments for model, effort and speed. Depth is now a per-turn setting the user changes on the fly — a line in `AGENTS.md` or a subagent body that tries to set depth ("think harder on migrations") competes with that setting instead of replacing it. Name the task's difficulty; leave depth to the knob.

**2. Outcome-first description.**

Codex subagents inherit GPT's outcome-first preference. The body should say what the agent produces, not how it works. Compare:

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

## Codex skills (SKILL.md in `.agents/skills/`)

Codex skills are conceptually similar to Claude skills — a `SKILL.md` with `name` + `description` frontmatter plus markdown body. Current docs: *"For repositories, Codex scans `.agents/skills` in every directory from your current working directory up to the repository root."* User scope is `$HOME/.agents/skills`, admin scope `/etc/codex/skills`, plus skills bundled with Codex. The December 2025 launch notes used `.codex/skills` / `~/.codex/skills`; the current docs list only `.agents/skills`, and whether the old path still loads is not stated — move skills found there. *"If two skills share the same `name`, Codex doesn't merge them; both can appear in skill selectors."*

### Frontmatter

Required: `name`, `description`. Same wording rules as Claude skills (`../artifacts.md` § SKILL.md → Description).

### Body

Same body rules as Claude skills — open with what + why, "When to use" section that repeats triggers, lean numbered workflow, output format, "When NOT to use" section, references files in a `references/` folder.

### Codex-specific notes

**1. The skill list has a budget — descriptions are cut first.**

*"By default, this list uses at most 2% of the model's context window, or 8,000 characters when the context window is unknown. If many skills are installed, Codex shortens skill descriptions first. For large skill sets, Codex may omit some skills from the initial list and show a warning."* (`skills.max_context_tokens` raises it, up to 10,000 tokens.) The docs' wording rule follows from that: *"write concise descriptions with clear scope and boundaries. Front-load the key use case and trigger words so a host can still match the skill if descriptions are shortened."* A description whose trigger phrases sit in its last sentence is a finding in a repo with many skills.

**2. Description must work for Codex's delegation heuristic.**

Codex picks a skill implicitly when the task *"matches the skill `description`"*, or explicitly via `/skills` or a `$skill-name` mention. Empirically, Codex is somewhat more conservative about auto-invoking than Claude Code. Lean toward concrete trigger phrasing and explicit "use whenever..." cues.

**3. A skill that never triggers may be switched off, not badly worded.**

Two config switches look like a wording failure: `[[skills.config]]` with `enabled = false` in `~/.codex/config.toml`, and `policy.allow_implicit_invocation: false` in the skill's `agents/openai.yaml` (explicit `$skill` invocation still works). Check both before rewriting the description — flag it, don't try to solve it with wording.

**4. Cross-vendor skills are possible.**

A `SKILL.md` written generically (no Claude-specific or Codex-specific assumptions) can be reused across vendors with a symlink or a copy (*"Codex supports symlinked skill folders"*). When reviewing such a skill, apply the cross-vendor wording rules from `../models/gpt.md`.

---

## Codex custom prompts and `/presets` — reusable prompts are skills now

*"Custom prompts are now deprecated. Use skills for reusable instructions and workflows instead."* (Codex changelog, 2026-01-22; the docs index still lists the page as *"Deprecated. Use skills for reusable prompts"*.) A reusable-prompt file reviewed today should become a skill:

- **Body:** the old slash-command rules carry over — open with the task in 1–2 sentences, a short numbered workflow, the output format, keep it short (it loads on every invocation).
- **Description:** now it matters. A custom prompt ran only when typed; a skill can also trigger implicitly, so write the description by the skill rules above — or set `allow_implicit_invocation: false` to keep it explicit-only, like the old command.
- **Arguments:** this file used to describe a `$ARGUMENTS` placeholder for custom prompts; the skills page read 2026-10-31 documents no placeholder — the user's request is the input. Don't carry `$ARGUMENTS` into a skill body.
- **Commands that mutate state** (commits, PRs, deploys): include explicit confirmation wording. Codex's behavior on destructive operations depends on the approval mode, so don't assume the agent will pause:

> "Before pushing, summarize the diff in 3 lines and pause for user approval. Only proceed after explicit go-ahead."

(This is the action-vs-suggestion-mode lever from `../techniques.md` §9.)

**`/presets` is not an artifact type.** Codex CLI 0.164.0 (2026-10-23): *"Save reusable prompts and slash commands in ten numbered /presets slots, with a keyboard menu for quick access."* Read from that wording (no file format is documented), a preset is a saved prompt for the user to fire, not a file the agent discovers; wording rules are those of an ad-hoc prompt (`../artifacts.md` § Ad-hoc prompt). A workflow other people or agents should reuse belongs in a skill.

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
| Subagent `model:` field | Opus / Sonnet / Haiku | GPT-6.x / GPT-5.6 + `model_reasoning_effort` | Codex has an additional reasoning knob — use it before rewriting body |
| Skill location | `~/.claude/skills/<name>/SKILL.md` | `.agents/skills/<name>/SKILL.md` (repo, CWD up to root) / `~/.agents/skills/` | Wording rules identical; Codex's skill list has a budget and shortens descriptions first — front-load triggers |
| Reusable prompt / slash command | `~/.claude/commands/*.md` | custom prompts deprecated — use a skill; `/presets` saves prompts in the TUI | Convert command files to skills; the description now matters |
| Hooks | settings.json hooks | `hooks.json` / `config.toml [hooks]` | Wording-equivalent: same "wording vs hook" trade-off applies |
| Headless | one-shot CLI invocation | `codex exec` | Codex makes this a first-class mode; tune ad-hoc prompts for non-interactive execution |
| MCP | supported | supported (STDIO + HTTP) | MCP tool descriptions are the same wording surface on both sides |
| Reasoning depth control | `effort: low/medium/high/xhigh` | `model_reasoning_effort`, ladder per model (`none` on 6.1 Terra / 6 Sol / 6 Luna / 5.6; not on 6 Astra / 6.1 Sol); changeable mid-conversation since 0.165.0 | Both vendors prefer the parameter over wording; `none` is GPT-only |

When in doubt about a Codex artifact and there's no Codex-specific rule above, fall back to the Claude rule for the same artifact type — they're more often the same than different.

---

## Source notes

- Skill paths, skill-list budget, description rule, `[[skills.config]]`, `agents/openai.yaml` `allow_implicit_invocation`, symlinks: https://learn.chatgpt.com/docs/build-skills.md — read 2026-10-31 (raw).
- Custom prompts deprecated (2026-01-22); `.codex/skills` in the December 2025 skills launch note; Codex CLI 0.163.0–0.166.0 notes (mid-conversation effort, memory v2, `/presets`, inline `/model`): https://learn.chatgpt.com/docs/changelog (HTML, tags stripped) — read 2026-10-31 (raw). Docs index entry "Custom Prompts: Deprecated. Use skills for reusable prompts": https://learn.chatgpt.com/llms.txt — read 2026-10-31 (raw).
- GPT-5.5 / 5.4 / 5.2 / 5.3-codex retirement in Codex, GPT-6.1 Terra CLI version: https://learn.chatgpt.com/docs/models.md — read 2026-10-31 (raw).
- `AGENTS.md` discovery order and the 32 KiB `project_doc_max_bytes` default re-checked against https://learn.chatgpt.com/docs/agent-configuration/agents-md.md — read 2026-10-31 (raw); unchanged.
- Subagent frontmatter fields, hooks, `codex exec`, MCP: not re-read in this revision.
