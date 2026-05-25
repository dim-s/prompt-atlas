# Common to all agentic systems — AGENTS.md as cross-tool standard

`AGENTS.md` has emerged as a de-facto open standard for agentic-tool persistent context. Multiple tools read it (Claude Code, Codex CLI, Gemini CLI when present, and increasingly others). This file documents the **shared semantics** — the parts that are true regardless of which agent reads it.

Tool-specific overlays live in `agentic-systems/claude-code.md`, `agentic-systems/codex.md`, etc.

---

## What AGENTS.md is

A markdown file at a project root (or, in some tools, hierarchical at multiple levels) that holds project conventions, commands, and gotchas an agentic tool should follow when working on the project. Tool-agnostic by design.

The intent: write rules once, share across whatever agentic tool a contributor uses. A team that has Claude Code users, Codex CLI users, and Cursor users can keep one `AGENTS.md` instead of three drift-prone variants.

---

## Adoption snapshot (May 2026)

| Tool | Reads AGENTS.md? | Notes |
|---|---|---|
| Claude Code | Yes (via `@AGENTS.md` import or convention) | CLAUDE.md is the native file; AGENTS.md import is the bridge |
| Codex CLI | Yes — natively, hierarchically | Native primary file; hard 32 KiB cap |
| Gemini CLI | **Yes — natively when `settings.json` configures `context.fileName`** | `GEMINI.md` is default; configure `fileName: ["AGENTS.md", "GEMINI.md"]` for cross-tool reuse |
| Kimi Code (Moonshot) | Convention-following; not yet documented in detail | Acts Claude-Code-like for routed usage; vendor-native CLI behavior TBD |
| Cursor | `.cursorrules` is native; AGENTS.md support varies | Check current state when reviewing |
| Aider | `CONVENTIONS.md` is native; AGENTS.md not standard | Native file is similar in spirit |
| Windsurf | `.windsurfrules` native | |
| Roo Code, Cline, Continue | Tool-specific | |
| Claude Code Router / OpenCode / OpenClaw / Kilo Code (cross-vendor routers) | Inherit host conventions | See § Cross-vendor routers below — the host system prompt is the binding constraint |

**Implication for review:** if you see `AGENTS.md` at a project root with no vendor-specific siblings (`.codex/`, `.claude/`, `.gemini/`, or vendor overrides `AGENTS.openai.md` / `AGENTS.glm.md` / `AGENTS.deepseek.md`), treat as **cross-vendor** — it must work for whatever agent reads it. Tune for the strictest constraint set; see `../models/_universal.md`.

**Three-vendor cross-tool reality (May 2026):** Claude Code, Codex CLI, and Gemini CLI all support `AGENTS.md` (some natively, some via configuration). This is the strongest validation yet of `AGENTS.md` as the open standard for agentic-tool persistent context. Gemini CLI's native support via `settings.json` brings the count of major frontier-vendor tools reading `AGENTS.md` to **three**. See [gemini-cli discussion #1471](https://github.com/google-gemini/gemini-cli/discussions/1471) for Google staff's thought leadership on this.

---

## Cross-vendor routers (the GLM-route-through-Claude-Code pattern)

A separate ecosystem has emerged in 2025–2026: **cross-vendor routers** that translate a vendor-native client's API calls to a different model. The most prominent:

- **[Claude Code Router](https://github.com/musistudio/claude-code-router)** — runs Claude Code, routes the calls to GLM, Kimi, DeepSeek, or other OpenAI-compatible endpoints
- **OpenCode / Cline / Kilo Code / OpenClaw** — variant agentic harnesses that share the routing model
- **DashScope's OpenAI/Anthropic dual compat surface** — Alibaba's hosted Qwen exposes both OpenAI and Anthropic API specs, letting Codex CLI or Claude Code talk to it without a separate router

These setups introduce wording concerns the single-vendor case doesn't have:

### 1. The host system prompt is unavoidable

Routers route requests as-is — they don't strip the host's system prompt. Your `AGENTS.md` / `CLAUDE.md` content stacks **on top of** Claude Code's (or whichever host CLI's) baseline system content. For GLM specifically, this is **the** load-bearing concern — heavy host prompts measurably suppress GLM's reasoning judgment.

### 2. Routers may inject mitigation directives

Claude Code Router and similar projects inject `<reasoning_content>` markers and explicit "write detailed reasoning before answering" instructions to re-open GLM's thinking gate. **Your prompt should be compatible with these injections** — don't write conflicting reasoning-format instructions.

### 3. Tokens stack — keep your contribution small

Host system prompt + your `AGENTS.md` + router injections + your message all share the routed model's context budget. For GLM-routed setups, target **<4 KiB load-bearing** in your `AGENTS.md`.

### 4. MCP tool descriptions travel as-is

Tools defined in Claude Code or other hosts are passed to the routed model via the OpenAI tools-array convention. Tool description quality matters as much for routed-model performance as for native-model performance.

### 5. Identity behaviors differ across hosts and models

A prompt that runs cleanly on native Claude Opus 4.7 may misfire on the same Claude Code session routing to GLM-5.1 — different model, different identity quirks (GLM's "I am Claude" distillation artifact), different reasoning gates. **A `CLAUDE.md` is no longer automatically a Claude prompt** in 2026 — it's a Claude Code prompt that runs on whichever model the user has routed.

### Review implication

When the user mentions running a router setup (or you observe one — `~/.claude-code-router/`, OpenCode-style configs, mentions of "I'm using Claude Code with GLM"):

1. **Verify the routed model in Step 2b** — don't assume Claude just because `CLAUDE.md` is the filename
2. **Load the routed-model file**, not Claude's — `models/glm.md`, `models/kimi.md`, etc.
3. **Apply the routed-model anti-patterns**: for GLM, the thinking-gate suppression is the headline; for DeepSeek-routed setups, user-prompt-priority is the headline; etc.
4. **Total token budget = host + your content + injections** — be conservative

Vendor-specific override files (`AGENTS.glm.md`, `AGENTS.deepseek.md`, `AGENTS.openai.md`) are the cleanest pattern for managing this — they let the user keep a shared baseline `AGENTS.md` and layer model-specific tweaks per route. Suggest the pattern when you see a single `AGENTS.md` trying to be everything to every routed model.

---

## Universal AGENTS.md rules

These hold regardless of which tool reads it. Tool-specific additions layer on top.

### 1. Markdown only — no required frontmatter

Plain markdown with headings. Don't wrap in YAML frontmatter unless a specific tool requires it (Codex doesn't parse YAML on AGENTS.md, and the user just sees `---\nname: ...\n---` as the first content block).

### 2. Short and specific

Every line earns its place. Test: *"If I delete this line, will an agent start making mistakes?"* If clearly no, propose removing.

Target size: **under 8 KiB** for the file's load-bearing rules. Reserves headroom for whatever combined context the tool computes (Codex's 32 KiB cap; Claude Code's soft context-rot threshold).

### 3. Concrete commands and constraints, not abstractions

`Run pnpm test:unit -- --changed before every commit` beats `run tests`.

`Use snake_case for Python files` beats `consistent naming`.

### 4. Explain WHY for non-obvious rules

`We use snake_case here because the older half of the codebase does and mixing looks noisy` is actionable; `use snake_case` is a constraint without context. The model generalizes from the reason.

### 5. What to include (universal)

- Bash commands the agent can't guess (custom scripts, non-standard test invocations)
- Code style differing from language defaults
- Testing instructions and preferred runners
- Repo etiquette (branch naming, PR conventions, commit messages)
- Architectural decisions specific to this project
- Environment quirks (required env vars, custom setup steps)
- Common gotchas / non-obvious behaviors
- Negative scopes ("Do not touch `legacy/` without explicit request")

### 6. What to exclude (universal)

- Anything derivable from reading the code
- Standard conventions any agent already follows
- Detailed API docs (link to them instead)
- File-by-file descriptions
- Information that changes frequently (sprint goals, in-flight migrations)
- Long explanations or tutorials
- Aspirational statements ("write great code")

### 7. Load-bearing invariants at edges, not middle

Edges of a file get attended to more reliably than the middle as context fills. Reference material can live in the middle; rules you need obeyed belong at top or bottom.

### 8. Don't pin specific tool/model names

`When using Opus 4.7, do X` or `Codex CLI users should Y` defeats the cross-tool purpose. Use functional descriptions: `When generating code, do X. When reviewing, do Y.`

### 9. Personal preferences go in a personal file, not the shared AGENTS.md

Repo-root `AGENTS.md` should hold project rules any contributor's agent would want to follow. Personal style ("speak Russian to me", preferred verbosity) goes in:
- `~/.claude/CLAUDE.md` (Claude Code, global personal)
- `~/.codex/AGENTS.md` (Codex, global personal)
- `CLAUDE.local.md` / `.codex/AGENTS.local.md` (per-project personal, gitignored)

If you see personal preferences in a shared `AGENTS.md`, propose moving them out.

### 10. The `.override.md` pattern (Codex; use cautiously elsewhere)

Codex supports `AGENTS.override.md` next to any `AGENTS.md` to layer overrides. Other tools may or may not. If you see an `.override.md` in a non-Codex context, confirm the tool actually reads it before assuming the override works.

---

## Cross-vendor AGENTS.md (the strictest case)

A bare `AGENTS.md` at project root meant to be read by multiple agents. Apply the strictest constraint along each axis:

| Axis | Strictest rule | Reason |
|---|---|---|
| Length | Under 8 KiB load-bearing | Codex's 32 KiB hard cap |
| Step prescription | Outcome-first; steps as "typical sub-tasks, choose order yourself" | GPT-5.5 treats step prescription as noise |
| Few-shot examples | Skip unless format genuinely strict | GPT-5.5 reasoning models are hurt by them |
| Persona / role | **CONTRADICTION** — Gemini wants persona (+5%), GPT-5.5 hurts | Surface to user; see SKILL.md § Contradiction detection. Compromise: methodological anchors only ("Apply systems-thinking lenses..."), drop credential-naming |
| Temperature in body | Don't reference temperature | Gemini fixed at 1.0; mentioning is dead weight |
| Tool guidance | Inside tool description, not AGENTS.md | OpenAI's strong split |
| Reasoning prompts | Never "think step by step" in body | All three vendors prefer the API parameter |
| CoT scaffolding | Strip entirely | Hurts on Gemini 3 and GPT-5.5; tolerated on Claude (so safe to remove) |
| Aggressive emphasis | Reserve for safety invariants only | Claude 4.5+ overtriggers |
| Output format | `json_schema` if downstream supports it | All three vendors prefer over prose constraints |
| Negative constraints | Place at end of file | Gemini drops early negatives |
| Structure mixing | Pick XML or Markdown, not both | Gemini stricter than Claude/GPT here |
| Blanket "do not" | Replace with positive scoped instructions | Gemini over-indexes on blanket negatives |

Full cross-vendor rules in `../models/_universal.md`.

**The unique three-vendor problem:** persona blocks. There's no neutral position — adding helps Gemini, hurts GPT-5.5, neutral on Claude. The `/prompt-atlas` skill should detect this and ask the user which target(s) before recommending strip-or-keep. See SKILL.md § Contradiction detection.

---

## Symptoms → wording fixes (universal)

| Symptom | Likely cause |
|---|---|
| Rule in repo-root AGENTS.md ignored | File too long; rule got buried. Or in Codex: combined hierarchy past 32 KiB |
| Rule followed only sometimes | Abstract language ("be careful with X"). Rewrite as concrete constraint with example |
| Agent contradicts a rule | Hedged wording ("maybe don't..."). Use declarative imperative |
| Agent asks about something the file answers | Rule is ambiguously worded. Rewrite directly |
| Agent applies a rule too broadly | "CRITICAL:" / "ALWAYS" stacking on Claude 4.5+. Drop emphasis |
| Tool rule ignored despite being relevant | Tool description is too vague (move guidance into tool description, not AGENTS.md) |

---

## CLAUDE.md ↔ AGENTS.md pattern (recommended)

When a project uses both Claude Code (with its native `CLAUDE.md`) and other agents:

```markdown
# CLAUDE.md
@AGENTS.md

## Claude-specific
- [rules that apply only when Claude is the agent]
```

`AGENTS.md` holds the portable rules. `CLAUDE.md` imports it via `@AGENTS.md` and adds Claude-specific deltas. **Don't duplicate** — duplicated rules drift.

The same pattern works inverted for Codex projects with a Claude side: native `AGENTS.md` holds shared rules, `~/.codex/AGENTS.md` adds personal Codex preferences.

---

## What goes in agentic-systems/<system>.md vs here

This file (`_common.md`) holds rules that are true **regardless of which agent reads AGENTS.md**.

Tool-specific files cover:
- File path conventions specific to that tool
- Hierarchy rules (Codex's hierarchical AGENTS.md, Cursor's single `.cursorrules`)
- Frontmatter quirks (Claude Code skills' YAML, Codex commands' `argument-hint`)
- Hooks model
- Headless / non-interactive mode quirks
- Hard caps (Codex's 32 KiB)
- Native non-AGENTS.md files (Cursor's `.cursorrules`, Aider's `CONVENTIONS.md`)

When in doubt: if the rule is about HOW to write the text, it belongs here or in `principles.md`. If it's about WHERE the text lives or HOW the system loads it, it belongs in `agentic-systems/<system>.md`.
