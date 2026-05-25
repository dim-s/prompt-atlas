# Gemini CLI — agentic system specifics

How Google's Gemini CLI (`gemini-cli`) layers on top of Gemini 3.x models. This file covers the **system**, not the **model** — for model behavior, see `../models/gemini.md` and `../matrix.md`.

Read this file when:
- The artifact is a `GEMINI.md` file, or `AGENTS.md` configured for Gemini CLI
- The artifact is under `~/.gemini/`, `.gemini/`, or in a project that uses Gemini CLI
- The user mentions Gemini CLI, `gemini` command, `/memory`, GEMINI.md, or Gemini agent skills
- Cross-vendor `AGENTS.md` review where Gemini CLI is one of the targets

Sources merged here: official Gemini CLI docs (`geminicli.com/docs`, `google-gemini.github.io/gemini-cli`), `gemini-cli` GitHub discussions on AGENTS.md, official Gemini API docs.

---

## GEMINI.md — the persistent-context file

### What GEMINI.md is

Gemini CLI's native persistent-context file, conceptually equivalent to Claude Code's `CLAUDE.md` and Codex CLI's `AGENTS.md`. The CLI reads `GEMINI.md` at session start and includes it in every prompt sent to the model.

Official: *"Context files, which use the default name GEMINI.md, are a powerful feature for providing instructional context to the Gemini model. You can use these files to give project-specific instructions, define a persona, or provide coding style guides to make the AI's responses more accurate and tailored to your needs."*

### Hierarchy

Hierarchical loading — multiple files concatenated top-down:

| Location | Scope |
|---|---|
| `~/.gemini/GEMINI.md` | All Gemini CLI sessions, globally |
| Workspace project root `GEMINI.md` | This project |
| Parent directories (above workspace root) | Picked up automatically |
| Workspace subdirectories | Layered when working in those subtrees |

The CLI *"loads various context files from several locations, concatenates the contents of all found files, and sends them to the model with every prompt."*

### Custom filenames — including `AGENTS.md` support

Critical for cross-tool work: Gemini CLI supports **custom filename configuration** via `settings.json`:

```json
{
  "context": {
    "fileName": ["AGENTS.md", "CONTEXT.md", "GEMINI.md"]
  }
}
```

This means **`AGENTS.md` is a first-class context file in Gemini CLI** when configured. With this setting, a single `AGENTS.md` at a project root is loaded by:
- **Claude Code** (via `@AGENTS.md` import in CLAUDE.md)
- **Codex CLI** (natively, hierarchical)
- **Gemini CLI** (natively when `settings.json` configures fileName)

This validates `AGENTS.md` as the **de-facto cross-tool standard** for agentic-tool persistent context.

### `@import` syntax

Same as Claude Code — Gemini CLI supports `@file.md` imports in GEMINI.md:

> *"You can break down large GEMINI.md files into smaller, more manageable components by importing content from other files using the @file.md syntax. This feature supports both relative and absolute paths."*

When reviewing a long GEMINI.md (>200 lines), propose splitting via `@import`. Same anti-bloat pattern as Claude Code.

### `/memory` commands

Gemini CLI exposes memory inspection and management commands:

| Command | Purpose |
|---|---|
| `/memory show` | Display the full concatenated context being sent to the model |
| `/memory add <text>` | Append text to global `~/.gemini/GEMINI.md` |

When troubleshooting "rule isn't being applied" in Gemini CLI, `/memory show` is the diagnostic tool — equivalent to checking which CLAUDE.md / AGENTS.md files are loaded in other vendors.

### Wording rules for GEMINI.md

Same universal AGENTS.md rules from `_common.md`:
- Short and specific (under 8 KiB load-bearing target — same cross-vendor budget)
- Concrete commands, not abstractions
- WHY for non-obvious rules
- Load-bearing invariants at edges, not middle

**Plus Gemini-3-specific overlays from `../models/gemini.md`:**
- **Add identity-based persona** at the top — "You are a senior backend engineer working in this codebase" — measurable boost on Gemini 3
- **Negative constraints at the end** — Gemini drops early negatives
- **Pick XML or Markdown, not both** — Gemini is stricter than Claude/GPT here
- **Don't reference temperature** — Gemini requires 1.0; mentioning in body is dead weight

### Cross-vendor GEMINI.md = AGENTS.md

If the project has both Claude Code and Gemini CLI users:
- Use `AGENTS.md` as the shared file (configure Gemini CLI's fileName to include it)
- Don't write duplicated `GEMINI.md` and `CLAUDE.md` — they will drift
- For Gemini-specific rules that don't apply to Claude, use the `@import` pattern: `AGENTS.md` imports a small `gemini-only.md` block

---

## Skills (`/skills`)

Gemini CLI supports skills similar to Anthropic's Agent Skills system. Documented in the CLI as `Agent Skills`.

### Frontmatter and body

Gemini CLI skills follow the same pattern as Claude Code skills:
- `name` and `description` in frontmatter
- Markdown body
- Skills can include reference files for progressive disclosure

Wording rules are the same as in `../artifacts.md` § SKILL.md and `claude-code.md` § SKILL.md.

### Description rules — same as Claude Code

The description is the trigger. Concrete verb, "use when...", "use proactively" if undertriggering, scope boundary if a sibling skill exists.

### Body rules — same plus Gemini-specific

Apply universal SKILL.md body rules + the Gemini-3-specific patterns from `../models/gemini.md`:
- Identity-based persona in the body (when the skill represents a "kind of thinker")
- No CoT scaffolding
- Negative constraints at the end
- Pick XML or Markdown

### Cross-vendor skills

A skill written for cross-tool reuse (Claude Code + Codex + Gemini CLI) should:
- Use `AGENTS.md`-compatible wording (no vendor-specific assumptions)
- Apply the cross-vendor compromises from `../models/_universal.md`
- Be especially careful about persona blocks — Gemini wants them, GPT-5.5 doesn't (see contradiction-detection in SKILL.md)

---

## Subagents — local and remote

Gemini CLI supports two subagent flavors:

| Flavor | Purpose |
|---|---|
| **Subagents** | Local task delegation — equivalent to Claude Code subagents |
| **Remote subagents** | Distributed task delegation across multiple agents |

### Wording rules

Same as Claude Code subagents (see `claude-code.md` § Subagents):
1. One-sentence functional role
2. Numbered workflow (5-8 steps)
3. Output format section
4. Priority-tagged findings if listing
5. Explicit don'ts where they matter

### Gemini-specific

For subagents running on Gemini 3.x:
- **Add identity-based role line at top** — "You are a [planner / reviewer / analyst]"
- **No CoT scaffolding in body** — `thinking_level` handles it
- **Persona will be obeyed strongly** — verify persona doesn't conflict with task constraints (gotcha in `../models/gemini.md` rule #5)

### Frontmatter — model selection

Subagent definitions can pin a specific Gemini model variant. The body's wording must be tuned for that variant:
- `gemini-3.1-pro-preview` — deep reasoning; less few-shot needed
- `gemini-3-flash` — balanced; few-shot for format helps
- `gemini-3.1-flash-lite` — bounded scope; concise mandatory

---

## Slash commands

Gemini CLI uses slash commands extensively (`/model`, `/memory`, `/settings`, custom commands).

### Wording rules

Same as Claude Code / Codex slash commands (see `../artifacts.md` § Slash-command prompts):
1. Open with task, 1-2 sentences
2. Reference `$ARGUMENTS` in context, not bare
3. Numbered workflow
4. Output format
5. Keep short — body loads on every invocation

### Gemini-specific notes

- Slash commands run in the user's current context — don't add identity-based persona in the body if the global `GEMINI.md` already has one. Avoid double persona.
- For commands with side effects (commits, deploys), include explicit verification step — same rule as Claude Code's "highest-leverage" advice.

---

## System prompt override

Gemini CLI configuration includes *"System prompt override"* — users can customize base instructions for model behavior beyond defaults.

Wording implication: when reviewing a Gemini CLI artifact, ask whether a system prompt override is in effect. Rules in `GEMINI.md` may be redundant or contradicted by an override.

---

## Auto Memory (experimental)

Gemini CLI has an experimental *"Auto Memory"* feature (marked with 🔬 research indicator) — automatically updates context based on session history.

This is similar to Claude Code's `/memory` flow. When reviewing Gemini CLI artifacts that rely on persistent learning across sessions, auto memory may be the right place rather than hand-edited GEMINI.md additions.

---

## Headless mode

Gemini CLI supports non-interactive invocations for CI / automation use.

### Wording implications

Same as Codex's `codex exec` and Claude Code's `claude -p`:
- **No interactive clarification** — front-load every decision
- **Strict output format** for downstream parsing — use `response_json_schema` if output is consumed by a script
- **Exit-code contract** for blocked operations rather than "ask user" wording

---

## Gemini CLI vs Claude Code vs Codex CLI — wording-relevant deltas

| Aspect | Gemini CLI | Claude Code | Codex CLI |
|---|---|---|---|
| Persistent context file | `GEMINI.md` (or `AGENTS.md` via config) | `CLAUDE.md` (with `@AGENTS.md` import) | `AGENTS.md` (native, hierarchical) |
| Hierarchy | Global → workspace → parents (concatenated) | Global → root → child dirs (on-demand for child) | Global → root → subdirs (`.override.md` pattern) |
| `@import` syntax | ✅ | ✅ | Not standard |
| `AGENTS.md` support | ✅ via `settings.json` fileName | ✅ via `@AGENTS.md` import | ✅ native |
| Hard cap | None documented (soft) | None (soft, ~300 lines) | 32 KiB (hard, silent drop) |
| Memory commands | `/memory show`, `/memory add` | `/memory` | — |
| Subagents | ✅ + remote subagents | ✅ + isolation: worktree | ✅ |
| Skills | ✅ (Agent Skills) | ✅ | ✅ |
| Slash commands | ✅ | ✅ | ✅ |
| Reasoning depth knob | `thinking_level` (minimal/low/medium/high) | `effort` (low/medium/high/xhigh) | `model_reasoning_effort` (none/.../xhigh) |
| Temperature | **Don't tune (1.0 fixed)** | Tunable | Tunable |
| Persona / identity prompting | **+5% boost — keep** | Neutral / OK | Hurts on 5.5 — strip |
| MCP support | ✅ | ✅ | ✅ |

Cross-vendor `AGENTS.md`: target the strictest constraint along each axis. Most strict cells:
- **Hierarchy hard cap** — Codex's 32 KiB
- **Persona** — opposite defaults Gemini vs GPT-5.5 (contradiction; see SKILL.md § Contradiction detection)
- **Temperature** — Gemini's "don't tune" wins
- **Negative constraint position** — Gemini's "at the end" wins
- **XML+Markdown mixing** — Gemini's "pick one" wins

---

## Symptoms → wording fixes (Gemini CLI-specific)

| Symptom | Probable cause |
|---|---|
| Rule in GEMINI.md not applied | Run `/memory show` to verify which files loaded; check hierarchy layering |
| `AGENTS.md` ignored despite being in project | `settings.json` doesn't configure fileName to include it; or wrong location |
| Subagent description doesn't trigger | Same recipe as other vendors — concrete verb, "use proactively / use when...", scope boundary |
| Skill doesn't trigger | Description vague + Gemini's slightly-conservative auto-invoke; lean toward more concrete trigger phrasing |
| Persona overrides task instructions | Gemini gotcha #5 — soften persona OR move task constraints into System Instructions |
| Mixed XML+Markdown errors out structure parsing | Gemini stricter than Claude/GPT — pick one |
| `thinking_level` and `thinking_budget` 400 error | Cannot coexist; migrate to `thinking_level` only |

---

## Common Gemini CLI anti-patterns

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **GEMINI.md without persona** | No identity-based system instruction | Add 1-line "You are a [role]" — Gemini-specific +5% boost |
| **GEMINI.md duplicating CLAUDE.md** | Same project has both files with overlapping rules | Consolidate into shared `AGENTS.md`, configure both CLIs to read it |
| **CoT scaffolding in GEMINI.md** | "Think step by step before answering" type rules | Strip; rely on `thinking_level` |
| **Temperature mention in body** | "Use temperature 0.3 for deterministic output" | Remove; Gemini default 1.0 is fixed |
| **GEMINI.md > 8 KiB load-bearing** | File is large with mixed concerns | Split via `@import` |
| **Negative constraints at top** | Rules like "Don't do X" in first paragraph | Move to end of file |
| **Mixed XML + Markdown** | `<context>` tags + `## Headings` in same file | Pick one |
| **Personal preferences in shared `AGENTS.md`** | "Speak Russian to me" in repo-root file | Move to `~/.gemini/GEMINI.md` |
| **Remote subagent without explicit context** | Body assumes parent's context but runs in isolation | Front-load all needed context |
