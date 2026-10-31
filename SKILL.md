---
name: prompt-atlas
description: Review and improve the WORDING of agent-facing or task-facing prompts. Covers frontier models — Claude (Fable 5.1, Opus / Sonnet / Haiku 5.5), GPT-6 / 6.1 and 5.6, Gemini 4 Argon and 3.x Flash, Kimi, GLM, frontier Qwen, DeepSeek, Grok, Mistral Large 4, Meta Muse (Spark / Wonder / Glimmer), Tencent Hunyuan, MiniMax / MiMo / StepFun — and small local 2-9B models (Gemma, small Qwen, Ministral, Phi, Llama, Granite, MiniCPM, LFM, Nemotron, RU tunes). Trigger when reviewing CLAUDE.md / AGENTS.md / GEMINI.md / SKILL.md / subagents / slash-commands / vendor overrides (AGENTS.deepseek.md etc.) / `suites/<name>/system*.md`. Russian — "проверь мой промпт", "улучши", "адаптируй под <модель>", "почему модель игнорирует / подменяет / не думает". English — "review my prompt", "tune for <model>", "why subagent isn't triggering", "Claude Code router prompt". Focuses on word choice, framing, structure, trigger heuristics — NOT on API params, SDK code, or file layout.
---

# Prompt Tuner

**Methodology:** apply the matrix-citation method — every finding cites a model × axis cell from `references/matrix.md` (or a section of a model file), so every recommendation is auditable and the user can challenge it from data, not authority.

Review and improve **how agent-facing prompts are written** — the words, framing, and structure that steer the model's behavior. A complete review has two halves:
1. **What's in the text and written poorly** (Findings)
2. **What's absent but should be there** given the prompt's purpose (Gap analysis) — often the higher-impact half.

This skill deals with text wording, NOT with API parameters, code, or where files live. (It surfaces non-wording levers like "raise `effort` before rewriting the prompt" as conversation points.)

## Coverage

Two target classes — the workflow branches on which one applies (Step 2a). **The current model lineup — ids, release status, reasoning knob, and the section to read — is in `references/lineup.md`.** This file never lists versions in detail; when a version matters, look it up there.

### Class 1 — Frontier agent-facing prompts

| Surface | Typical artifacts |
|---|---|
| Claude / Claude Code | CLAUDE.md, AGENTS.md, SKILL.md, `.claude/agents/*.md`, `.claude/commands/*.md` |
| OpenAI / Codex | AGENTS.md, AGENTS.override.md, `.agents/skills/` (older: `.codex/skills/`), `.codex/agents/`, `codex exec` prompts |
| Google / Gemini CLI | GEMINI.md, AGENTS.md (when configured), `.gemini/skills/`, `.gemini/agents/`, `.gemini/commands/`, headless prompts |
| Other frontier vendors (Kimi, GLM, frontier Qwen, DeepSeek, Grok, Mistral, Meta Muse, Tencent Hy, MiniMax / MiMo / StepFun) | system prompts over the vendor API, AGENTS.md / CLAUDE.md routed through Claude Code Router / OpenCode / Cline / Kilo Code / Cursor / OpenClaw, vendor overrides (`AGENTS.glm.md`, `AGENTS.deepseek.md`) |

Frontier Qwen only — small Qwen variants are Class 2. DeepSeek's user-prompt-priority makes vendor overrides more common there than elsewhere.

### Class 2 — Small local model task prompts

System prompts for models run locally on a single-pass task (extract / classify / NL→DSL / route / score): dense 2–9B, or MoE with ≤ ~9B active and ≤ ~35B total (one consumer GPU / a 32–64 GB Mac). Typical locations: `suites/<name>/system.md` and `system_<model>.md`, `prompts/<topic>/<hypothesis>.md`, LM Studio / llama.cpp / Ollama / vLLM / MLX system prompts, inline `--system` strings. Different axes apply — `references/matrix-small.md`.

### Out of scope (both classes)

Parameter *value* tuning (`temperature`, `max_tokens`, `effort` / `reasoning_effort` / `thinking_level` / `thinking` toggles), vendor `config.toml` / `settings.json`, MCP registration, hook scripts, SDK code, LoRA / SFT data design — those belong to `claude-api`, vendor docs, `update-config`, or project docs. Also out of scope: **decision models** (state + typed questions in, calibrated probabilities out — no system prompt to word) and speech-to-speech voice models other than the one voice section in `models/gpt.md`. Legacy models live in `references/archive/` — load it only when a prompt still targets one.

### Reasoning-depth knobs — surface, do NOT embed

Every current frontier vendor exposes reasoning depth as a **runtime parameter** set out-of-band (Claude `effort`, OpenAI / xAI / GLM / Qwen / Kimi / DeepSeek / Mistral / Meta `reasoning_effort` or `reasoning.effort`, Gemini `thinking_level`, …). The per-model ladder, default and whether thinking can be switched off at all are in `references/lineup.md`.

The model **cannot change its own reasoning depth from prose.** Lines like `Use effort: xhigh for this task`, `Think at high level`, `Set thinking_level to medium`, pasted into CLAUDE.md / AGENTS.md / GEMINI.md / SKILL.md / a subagent body / a slash-command body are **inert text** — the same failure mode as "think step by step" (`antipatterns.md` #31). On several current models the vendor has *measured* that "think less / answer directly" lines don't reduce thinking (Sonnet 5.5, Haiku 5.5), and where thinking cannot be disabled such lines are unimplementable.

When raising or lowering effort would help, **say it to the user as a conversation point** — never as an `[ADD]` snippet for the file:

> ✅ "If you're running this in Claude Code, try `/effort xhigh` before we rewrite anything — it often restores the behavior you want and costs nothing."
> ❌ `[ADD]` to CLAUDE.md: `Use effort: xhigh for kernel work, medium for brainstorm.`

**`effort` does not control visible length** on the Opus 5 lineage — a verbosity complaint there is a wording fix (`techniques.md §19`), not "drop to `medium`". Effort remains the right first lever for *depth* complaints.

**Exceptions — places where a knob legitimately lives in the artifact:**
1. **Subagent frontmatter.** Claude Code `.claude/agents/*.md` `effort:` (`low`…`max`, overrides the session effort; also `--agents` JSON / Agent SDK `AgentDefinition.effort`; a session-wide default for subagents is the `CLAUDE_CODE_SUBAGENT_EFFORT` env var, not prose). Codex `.codex/agents/*.md` `model_reasoning_effort:`. Config, not prose — propose it when pinning a per-agent effort is the goal.
2. **GLM reasoning re-injection** (`<reasoning_content>` markers) under heavy router prompts — a documented mitigation, but on GLM-5.2+ a *fallback* behind `reasoning_effort` where the router forwards it (`models/glm.md` § Family-wide rules #1).
3. **Muse Spark Open** — Meta documents a `Reasoning strength: N.` line in the system prompt that overrides the request parameter (`models/meta.md`). A vendor-defined control string, valid only for that model; not a precedent for other vendors.
4. **Claude Code `ultrathink`** typed in a user prompt runs that turn at `xhigh` (Claude Code 2.1.311+) — a CLI keyword, not wording guidance; don't write it into persistent files expecting the same effect.

## When to auto-trigger

Trigger proactively whenever the user works with any artifact type above, even without an explicit "review my prompt". Typical cues:

- They opened, edited, pasted or asked to write one of the file types
- A subagent or skill "isn't triggering" or "triggers too often"
- They ask whether wording "looks right" / "is clear enough", or say "проверь", "улучши", "review", "audit", "почему не работает", "адаптируй под кодекс", "tune for <model>"
- **They migrated to a newer model** and want prompts updated — most current releases ship as "runs on the old prompts" plus a short delta list, often a *strip* audit (Opus 5 lineage, GPT-5.6) rather than an add one. The per-model checklists are in `references/model-audits.md`
- A model "suddenly" got wordier / terser, narrates more or less, delegates more eagerly, re-checks its own work, stops early or asks permission mid-task — ask which version first; most of these are documented release defaults fixed by wording
- They switch agentic environment (Claude Code ↔ Codex CLI ↔ Gemini CLI) or route through Claude Code Router / OpenCode / Cline / Kilo Code / OpenClaw (most common: GLM-routed setups → `models/glm.md` § The Claude-Code-router pattern)
- A creative / narrative / advisory / coaching / brainstorm agent feels flat, mechanical, won't push back → `references/patterns.md` § creative-domain kernel
- Code-switching / anglicisms / language mixing in non-English output ("много англицизмов", "соскальзывает на английский", "теряет регистр") → `references/multilingual.md`
- They name a non-Anthropic vendor, host or version string (Moonshot, Z.ai / Zhipu, QwenCloud / DashScope, deepseek.com, x.ai, Mistral, Meta Model API, Tencent TokenHub, MiniMax, Xiaomi MiMo, StepFun …) → the vendor file via `references/lineup.md`
- A documented vendor failure: "GLM не думает в Claude Code / claims it's Claude" → `models/glm.md` rules #1, #3 · "Qwen скипает разделы" → `models/qwen-frontier.md` rules #2, #1 · "DeepSeek игнорирует system prompt / 400 on second turn / уверенно говорит, что X нигде нет" → `models/deepseek.md` rules #1, #3 · "Kimi swarm не вызывается" → `models/kimi.md` § Agent Swarms
- **Class 2:** they edit `suites/<name>/system.md` / `system_<model>.md` or any prompt aimed at a small local model (dense 2–9B, small MoE) → `references/matrix-small.md` + `models/small-local.md`; small-model symptoms ("подменяет / выдумывает / игнорирует правила", "не держит negation", "thinking leaks into output", "markdown fences keep leaking", "качество просело, когда добавил правила") → `references/antipatterns-small.md`
- "Works on Opus, fails on gemma / qwen / ministral" → load both matrices; treat as cross-target adaptation

When borderline, lean toward triggering — a short review costs little, a missed issue costs more.

---

## Prime directive — preserve hidden intent

Prompts encode load-bearing logic in ways that look like stylistic choices. A phrasing that seems verbose, awkward, or redundant is often there because the user saw a specific failure mode and chose those exact words to prevent it. **Rewriting blindly strips that protection** and causes a regression the user must re-diagnose from scratch.

Treat existing text as **evidence of past decisions**, not first-draft cruft. Before you propose removing or rewording anything:

1. **Ask: "Why might this have been written this way?"** If you can't generate a plausible reason, you don't yet understand the prompt — not that the line is useless.

2. **Signals a line is probably load-bearing** — handle with extra care:
   - Unusual phrasing, repetition, or emphasis (all-caps, "NEVER", "ALWAYS", "IMPORTANT") — often a scar from a specific incident
   - Negative instructions ("do NOT do X") — the opposite behavior almost certainly happened at least once
   - Specific examples, file paths, function names, numeric thresholds
   - Trigger phrases in SKILL.md / subagent `description` fields — every keyword may be a deliberate heuristic hook
   - Seemingly redundant clauses ("X — do not do Y") — the second clause is the fix, the first is context
   - Non-standard order or structure

3. **Default to the smallest change that fixes the finding.** A targeted edit of one phrase beats a rewrite of a paragraph. A rewrite of a paragraph beats a rewrite of the file. Only do a full rewrite when the user explicitly asks for one.

4. **If you don't know why a line exists, ask — don't delete.** List under "Assumptions / questions" and wait for confirmation. Losing one review cycle to clarification is cheaper than silently removing a rule that was preventing a bug.

5. **Shortness is a goal, not a mandate.** Trim only what you can explain *why* is safe to trim.

This matters most for files loaded into every session (CLAUDE.md, AGENTS.md) or used as delegation triggers (subagent / skill descriptions). The cost of breaking hidden logic is paid repeatedly, across every future conversation.

---

## The workflow

### Step 1 — Identify the artifact type

The same sentence is tuned differently depending on where it lives. Quickly classify:

- Heading + bullet list of project rules → **CLAUDE.md / AGENTS.md** (an `AGENTS.md` may belong to either vendor — see Step 2)
- YAML frontmatter with `name` + `description` + body, no `model:` field → **SKILL.md**
- YAML frontmatter with `name` + `description` + tools / `model:` field + body → **subagent**
- Markdown body in `.claude/commands/` → **slash-command prompt**. A Codex custom prompt (`~/.codex/prompts/`) is deprecated — review it as a skill to convert (`agentic-systems/codex.md` § Codex custom prompts)
- Free text the user wants to paste into a chat → **ad-hoc prompt**

If unclear, ask: "Is this a CLAUDE.md / AGENTS.md / a subagent / a skill / a slash command / something else?"

### Step 2 — Identify the target class, vendor, and model

#### Step 2a — Target class (frontier agent-facing vs small local task-facing)

This split decides which reference set applies. Get it right before the vendor question.

**Frontier (Class 1) signals:** file under `.claude/`, `.codex/`, `.agents/`, `.gemini/`, or a project-root `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `AGENTS.override.md`; frontmatter names a frontier model; the prompt steers a multi-turn, tool-using agent; the user mentions Claude Code / Codex / Gemini CLI or a router; the goal is judgement, taste, generalization. → Step 2b → 2c → Path A.

**Small local (Class 2) signals:** `suites/<name>/system*.md`; LM Studio (`~/.lmstudio/`), llama.cpp, Ollama, vLLM, MLX; a local model named (Gemma, small Qwen, Ministral 3 3B / 8B, Phi-4-mini, Llama 3.2, Granite, MiniCPM, LFM, Nemotron small / Dory, fine-tunes like saiga / T-lite / Hermes); a single-pass compile-time task with structured output (JSON, schema, GBNF); small-model failure symptoms (substitution bias, dropped adjectives, skipped negation, thinking-token leak, markdown emission, repetition loops). → skip 2b/2c → Path B.

**Mixed target:** one prompt for both classes, a migration between them, or a hardware-tier decision. → load both paths; flag diverging axes (`principles-tolerance`, `few-shot-density`, `position-of-critical`, `thinking-on`, `persona`, `system-language`).

**If unclear,** ask one short question (in the user's language, § Language):

> "Is this prompt for a frontier model running in an agentic CLI / API, or for a small local model (2–9B, like Gemma / Qwen / Ministral / Phi / Llama) on LM Studio / llama.cpp / Ollama?"

#### Step 2b — Vendor (frontier only)

Infer from signals; ask only if they conflict.

| Vendor | Signals (paths, names, hosts, parameters) |
|---|---|
| Anthropic | `.claude/`, `CLAUDE.md`, `model: opus \| sonnet \| haiku`, Claude / Fable / Mythos / Opus / Sonnet / Haiku, `claude-*` ids |
| OpenAI | `.codex/`, `.agents/skills/`, `AGENTS.override.md`, Codex, GPT, `gpt-6*` / `gpt-5.x` ids, `model_reasoning_effort`, `text.verbosity`, `phase`, `json_schema`, GPT-Live |
| Google | `.gemini/`, `GEMINI.md`, Gemini CLI, `thinking_level`, `response_json_schema`, `google-genai`, AI Studio / Vertex, `gemini-4-*` / `gemini-3.x-*` ids, Antigravity |
| Moonshot | Kimi, K3 / K3.1 / K2.x, `platform.kimi.ai` / `platform.moonshot.ai`, `preserve_thinking`, Agent Swarms, Kimi Code CLI |
| Z.ai | GLM, Zhipu, `zai-org`, `api.z.ai`, `clear_thinking`, ox-alpha, routed through Claude Code Router / OpenCode / Cline |
| Alibaba (frontier) | Qwen3.8-Max / Flash / Omni-Flash, QwenCloud, DashScope, Model Studio, `enable_thinking`, `preserve_thinking` |
| DeepSeek | DeepSeek, V4 / V4.1, `deepseek-flash`, `deepseek-v4-pro`, `reasoning_content`, `thinking` toggle, `output_config.effort`, DeepSeek Harness. Pinned `deepseek-chat` / `deepseek-reasoner` / `deepseek-v4-flash*` ids are a **migration finding** (retired or re-routed), not a version signal |
| xAI | Grok, `grok-4.x`, `x.ai`, Grok Skills, `guidelines` request parameter |
| Mistral | Mistral Large 4 / Large / Medium / Small, Ministral 14B, La Plateforme, `mistral.ai`, `mistral-large-4` (Ministral 3 3B / 8B → Class 2) |
| Meta | Muse Wonder / Spark / Spark Open / Glimmer, Muse Code (an agent, not a model), Meta Model API, `muse-wonder-*` / `muse-spark-*`, `meta.ai`, `Muse-Spark-Open-*`, `Muse-Glimmer-30B` |
| Tencent | Hunyuan, Hy4, TokenHub, WorkBuddy / CodeBuddy / Yuanbao / ima |
| Other frontier | MiniMax (M3.x, MiniMax Code), Xiaomi MiMo, StepFun Step → `models/other-frontier.md` |

**Cross-vendor `AGENTS.md`:** a project-root `AGENTS.md` with no vendor-specific sibling directories or overrides is genuinely cross-vendor — tune for the lowest common denominator. **Default to three-vendor (Claude + GPT + Gemini)** unless the user names another vendor; escalate to 4+ only on clear evidence (mixed-router setup, explicit statement). See `models/_universal.md` § Cross-vendor (4+) for the 4+ trap.

#### Step 2c — Model version (frontier only)

**Read `references/lineup.md`.** It lists, per vendor, the current and previous models with ids, release status, reasoning knob and the section to load, plus what moved to the archive. Two rules that the lineup does not replace:

- **Ask the version even when the vendor is obvious.** Inside the Claude family the recommendation flips by version on self-review, task checks, delegation, narration and the thinking off-switch (Step 4); inside GPT-6, `none` effort and sampling exist on some tiers and not others. "It's a Claude prompt" no longer determines the advice.
- **A version the lineup doesn't know is a gap to state, not to fill.** If the user names a model that isn't in the lineup or the archive, say the atlas doesn't cover it yet and review against the family rules — don't invent version deltas.

If you can't infer, ask one short question, in the user's language:

> "Which model is this prompt for — and on which surface (API, Claude Code, Codex, Gemini CLI, a router)? If it has to work on several, name them."

### Step 3 — Read the relevant references

**The references are the source of truth.** They are updated as vendors publish guidance and as new failure modes are observed. A review generated from memory locks the skill to months-old training data.

**Every review:** `references/principles.md` (vendor-neutral principles) — read first.

#### Path A — Frontier (Class 1)

1. `references/lineup.md` — which model, which knob, which section.
2. `references/matrix.md` — the row(s) for the target model(s). **Cite the cell** that drives each change.
3. `references/model-audits.md` — the migration / targeting checklist for the target model.
4. `references/models/<vendor>.md` — nuance the matrix can't hold (claude, gpt, gemini, kimi, glm, qwen-frontier — not small-local —, deepseek, grok, mistral-frontier, meta, tencent, other-frontier). Multi-model or cross-vendor prompts: `models/_universal.md` (the Universal-X section + the Cross-vendor section).
5. `references/artifacts.md` — the section for the artifact type from Step 1 (plus the "Codex variant" for Codex artifacts).
6. `references/antipatterns.md` — scan once; match findings to documented patterns rather than improvising.
7. `references/agentic-systems/<system>.md` — Claude Code, Codex, Gemini CLI; `_common.md` for cross-tool `AGENTS.md` semantics. Routers and hosted APIs without a file: use `_common.md` + the vendor file (e.g. GLM's router interference lives in `models/glm.md`).
8. `references/techniques.md` (on demand) — exact wording of frontier snippets. Don't paraphrase from memory — copy and adapt.

#### Path B — Small local (Class 2)

1. `references/matrix-small.md` — small-model axes (few-shot-density, position-of-critical, principles-tolerance, implicit-negation, EN-system unlock, thinking-impact, tools-impact, multi-pass-tolerance, substitution-bias, markdown-tolerance). Cite cells the same way.
2. `references/models/small-local.md` — family deltas; family quirks (Gemma no system role, Ministral markdown emission, Qwen substitution bias, chat-template thinking flags) override matrix defaults.
3. `references/techniques-small.md` — ready-to-paste skeleton and blocks. Copy and adapt.
4. `references/antipatterns-small.md` — scan once; each anti-pattern carries its reproduced regression size.

#### Path C — Mixed

Load A and B; use `principles.md` for the shared layer; highlight axes where they conflict (Step 4, cross-class table).

#### Always available

- `references/multilingual.md` — non-English prompt or output, or symptoms like "англицизмы", "code-switching", "теряет регистр". Both classes.
- `references/patterns.md` — the three install-once patterns (below).
- `references/archive/` — legacy models; only when the prompt still targets one.

### Step 4 — Detect contradictions before findings

Before producing findings, scan the prompt for axes where the recommendation **flips depending on the target model**. If the prompt touches such an axis and the target is not disambiguated, ask first (template below).

#### Across vendors — the notorious axes

1. **Persona / "You are X"** — Gemini 3.x wants it (+5%, Google); GPT-5.x hurts (measured on 5.5; GPT-6 guide silent); Claude neutral; Kimi helps (Moonshot guide); GLM functional-only, no identity pinning; frontier Qwen OK; DeepSeek brief-only; Gemini 4 Argon, Mistral Large 4, Meta, Tencent, other-frontier `?`.
2. **Sampling parameters in the design** — gone on every Claude (API version `2026-10-21`: 400 at any value); Gemini ignores them (400 announced); GPT-6 Astra / 6.1 Sol: unsupported, other GPT-6.x tiers only at effort `none` (`models/gpt.md`); Kimi K3.x and several open-weight cards fix sampling server-side. Treat sampling as unavailable for tone or variety — steer with wording; "propose N directions, then implement the chosen one" is the cross-vendor variety pattern.
3. **CoT scaffolding ("think step by step")** — hurts Gemini 3.x and GPT-5.x, tolerated on Claude, inert where depth is a parameter (Kimi, GLM, Qwen, DeepSeek, Grok, Mistral, Meta). Never the right lever on a current model.
4. **Length instructions ("be brief" / "be thorough")** point in **opposite directions**: the Opus 5 lineage runs long and effort won't shorten it; GPT-5.6, Gemini 3.6 Flash and DeepSeek V4.1-Flash-1017 (vendor: "shorter outputs") are terser than what they replaced, so a carried-over "be brief" overcorrects; Gemini 3.8 Flash works harder by design. Safe cross-vendor form: state length as a requirement of the deliverable ("at most 5 bullets"), not as a disposition.
5. **"Answer without reasoning" / "don't overthink"** — inert or harmful everywhere and **unimplementable where thinking cannot be disabled** (the lineup's knob column marks these: e.g. Claude Opus 5.5, Kimi K3.x / K2.7-Code, GLM-5.3 family, Grok 4.5+, Mistral Large 4, hosted Muse and Muse Spark Open, MiniMax M3.1-Flash, StepFun Step 5, the Qwen3.8-Max open weights; Sonnet 5.5's lowest mode is `between_tools`). Strip unconditionally; the only depth lever is the parameter.

#### Inside the Claude family

If the prompt touches any of these and the Claude version is unknown, ask:

| Axis | Fable 5.1 | Opus 5.5 (Opus 5 baseline) | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Self-review / "double-check your answer" | neutral | **strip — over-verifies** | not addressed | not addressed |
| Task check ("run a real check that exercises the change") | fine | fine (task verification) | **official snippet at `low`** | **official snippet at `low`/`medium`** |
| Subagent delegation wording | boundaries | boundaries + cap | stop self-started reviewer subagents at `xhigh`/`max` | ? |
| Narration between tool calls | quiet — don't suppress | narrates more — describe the shape | updates arrive as thinking blocks | ? |
| Thinking off-switch | none | none | `between_tools` (≤ `high`) | `disabled` (≤ `high`) |
| Forced `tool_choice` | 400 | 400 | 400 | accepted (no thinking block) |

The verification rows are the sharpest: a `[CRITICAL] strip self-review` on Opus 5.5 and an `[ADD] real-check paragraph` on Haiku 5.5 at `low` are both correct, because one is a ritual and the other is part of the task. Universal wording that survives the family: `models/claude.md § Universal` rule 7.

Less common: few-shot examples (help Claude / Kimi / frontier Qwen for format, hurt GPT-5.x reasoning, mixed on Gemini); negative-constraint position (Gemini drops early ones; Muse Spark Open "pays more attention to the beginning of the system prompt"); aggressive emphasis (overtriggers on current Claude); XML+Markdown mixing (Gemini strict).

#### Cross-vendor axes outside Claude / GPT / Gemini

| Axis | Direction by vendor | Recommendation |
|---|---|---|
| **Instruction placement (system vs user)** | **DeepSeek: user-prompt priority** per V4 practitioner guidance (citing R1-era vendor advice) — the ~85%-of-errors figure predates current builds and twice failed to reproduce (on V4-Flash-0731, now archived) | Authoring fresh for DeepSeek: bulk in the user prompt. Porting a working system-side body: measure before re-architecting (`models/deepseek.md` field notes) |
| **System prompt size** | **GLM: heavy host prompts suppress thinking** (Claude Code's prompt included) | GLM-routed: <4 KiB load-bearing; `reasoning_effort` first, re-injection as fallback |
| **Self-verification prompts** | **frontier Qwen: load-bearing for completeness**; Opus 5 lineage: strip | Qwen in scope: "review your response before finishing" — but not in a set that includes Opus 5.x without a condition |
| **Numbered requirements vs prose list** | **frontier Qwen skips un-emphasized prose sections** | Number critical requirements explicitly |
| **Identity pinning ("You are GLM-…")** | **GLM distillation artifact — may claim "I am Claude"**; MiniMax's open template injects its own identity line when no system prompt is given | Functional roles only in cross-vendor prompts |
| **XML context with `relevance` attrs** | **DeepSeek: load-bearing** (92% vs 45% reported) | Helps DeepSeek; safe elsewhere |
| **JSON demanded in prose AND `response_format`** | **DeepSeek: both required** | Demand JSON in the user prompt even with the API param set |
| **Granular constraints (hex colors, sizes, scopes)** | **frontier Qwen: vagueness hurts more** | Restore detail you might strip for GPT |
| **Agent Swarms / parallel sub-agent decomposition** | **Kimi only** (K2.6 lineage; K2 series retiring end of 2026) | Vendor-specific; don't write swarm prompts for portability |
| **Context window on "upgrade"** | **Grok 4.5+: 500K, down from 4.3's 1M** | Confirm version before long-context patterns |
| **Harness-side context management** | **Meta Muse Spark: the model manages its own 1M window** | Strip "summarize every N turns" scaffolding there; keep it where the harness owns trimming |
| **Moderation rules in prose** | **xAI `guidelines` request parameter** moves moderation policy out of the system prompt | On Grok, prefer the parameter; keep prose rules vendor-neutral elsewhere |
| **Persistent reasoning across turns** | Kimi explicit (`preserve_thinking`), Qwen explicit, DeepSeek (mandatory with tools) / Mistral Large 4 / MiMo via full-message replay; Muse Wonder: none on Chat Completions | Don't write prompts that depend on visible-reasoning continuity across turns |
| **Image content placement** | Put image blocks in user turns (`models/deepseek.md` for the vision-model history) | Never generate a system-side image block |

#### Cross-class — frontier vs small local

Apply when one prompt must work on both classes, or when migrating between them. These axes go in **opposite directions**:

| Axis | Frontier (Class 1) | Small local (Class 2) |
|---|---|---|
| Abstract principles in the body | tolerated, often helpful | regression (−7 to −15pp on 2-3B) — convert to few-shot demonstrations |
| Thinking / CoT on | helps (depth via the effort parameter) | regression (−25 to −44pp on 2-3B) — disable via the chat-template / provider flag where one exists; always-thinking models (LFM2.5-24B-A2B, Dory in vLLM): budget + strip (`antipatterns-small.md` § 2) |
| Hint-and-vibes prose (creative kernel) | helps on literal frontier models | regression — small models can't extrapolate from vibes; need literal demonstrations |
| Same-model self-verify | sometimes helps (and strip on the Opus 5 lineage) | regression (−40pp) — confirmation anti-bias |
| Persona block "You are an expert at X" | helps Gemini 3.x, neutral Claude, hurts GPT-5.x | mostly noise on 2-3B compile tasks |
| System role | first-class on frontier vendors | none on Gemma — fold into the first user turn |
| Native tool calling | first-class | regression on 2-3B without 3-5 few-shot tool-call examples |
| Long blacklists of forbidden behaviors | tolerated | regression (Pink Elephant) — whitelist + 3-5 named anti-patterns |
| Same-language system on a non-EN task | preferred | **inverted** — EN system unlocks +4-8pp on RU/DE input for 2-3B base models |
| Critical rules position | tolerated anywhere | at the END, after Forbidden |

**If the prompt has content on a contradicting axis AND the target isn't disambiguated, pause and ask** before producing findings. One clarification turn costs less than a wrong recommendation that breaks a working prompt.

#### Contradiction question template

Render in the user's language; keep the structure, model names and quoted snippets verbatim. Fill the bullets from the matrix cells for the models actually plausible here — the example shows the shape:

> ⚠️ **Before continuing, I need to clarify.** The prompt contains elements where the recommendation **depends on the target model**, and you haven't specified one:
>
> 1. **"Double-check your work before finishing"**:
>    - On Claude Opus 5.5 (Opus 5 baseline): **strip — over-verification** (Anthropic guide)
>    - On Claude Haiku 5.5 at `low` effort: **replace with the official "run a real check that exercises the change" paragraph**
>    - On frontier Qwen: **keep — load-bearing for completeness**
> 2. **Persona block** ("You are a senior engineer..."): Gemini 3.x keep (+5%), GPT-5.x strip, Claude neutral, GLM functional-only.
>
> Which model is this prompt for? If several, I can apply the strictest cross-vendor compromise (`models/_universal.md`).

For DeepSeek-suspicious prompts (large system prompt + complex instructions) add the instruction-placement contradiction from the table above.

#### When NOT to ask

- The vendor and version were identified unambiguously in Step 2 and there's no within-family contradiction
- The contradiction would change a `[POLISH]` finding only
- The user said "give me the universal compromise" — apply cross-vendor wording
- The artifact is tied to one vendor (`.claude/`, `.codex/`, `.gemini/`, `AGENTS.deepseek.md`, `AGENTS.glm.md`)

#### After clarification

Proceed to Step 5 and note which findings were contingent on the answer. If the user picks "cross-vendor / universal", apply the strictest compromise from `models/_universal.md` and cite it.

---

### Step 5 — Produce findings, gap analysis, then changes

#### Findings — what's broken in existing text

Tag each:
- `[CRITICAL]` — will cause the prompt to misfire (vague subagent description that never triggers, negative instructions with no alternative, missing verification on a side-effect command, an instruction the target model cannot implement)
- `[IMPROVE]` — will meaningfully improve behavior (missing WHY, no examples, negative framing, verbose rule list)
- `[POLISH]` — small wording cleanup

Each finding points to the exact line or phrase, with current behavior vs. expected behavior after the change. **When citing the matrix, name the row × column** that drove the finding.

#### Gap analysis — what's missing

1. **State the prompt's purpose in one sentence.**
2. **Enumerate plausible failure modes** for that purpose — the ones THIS prompt would plausibly hit.
3. **Cross-reference** `principles.md`, `matrix.md`, `model-audits.md`, `agentic-systems/<system>.md`, `techniques.md` for snippets addressing them.
4. **Propose each missing element as `[ADD]`** with (a) the failure mode prevented, (b) the source, (c) the exact snippet. Don't paraphrase — copy and adapt.

**When a target model is known, run its checklist in `references/model-audits.md` first** — on current releases the vendor's delta list (what to strip, what to add, what now returns 400) is usually most of the review.

**Common high-value additions by artifact type** (always justify from THIS prompt, not this list):

| Artifact | Often missing |
|---|---|
| Subagents with tool access | A task check for side-effecting work (`agentic-systems/claude-code.md` § Verification) — phrased as part of the task, never as a self-review ritual on the Opus 5 lineage (`antipatterns.md` #37); permission to express uncertainty (`principles.md` #11); output format with priority tags |
| Subagents meant for delegation | "Use proactively" / "Use when..." in description; negative scope boundary against the sibling agent |
| CLAUDE.md / AGENTS.md | WHY for non-obvious rules; concrete commands instead of abstractions; load-bearing invariants at edges; negative-scope section ("don't touch X") |
| SKILL.md descriptions | Explicit trigger clauses with file names / user phrasings; "Do NOT use when..." boundary; bilingual keywords if the domain is non-English. Claude Code truncates `description` + `when_to_use` at 1,536 characters in the skill listing and escapes angle brackets — front-load the triggers |
| Slash commands with side effects | Verification step; explicit reversibility wording; cleanup-after-yourself |
| Slash commands meant for skill behavior | `disable-model-invocation: true` if side effects (Claude Code) |
| Ad-hoc prompts | Functional role sentence; success criteria; one or two examples (Claude) or zero-shot + criteria (GPT) |
| Long-context / RAG prompts | Document at top / question at bottom; quote-grounding; permission to say "I don't know" |
| Agentic coding prompts | Anti-overengineering / scope snippet; anti-hard-coding; subagent-orchestration guidance |
| Unattended / async agents | The vendor's autonomous-run block where one exists (Fable 5.1, Opus 5.5 — `model-audits.md`); an explicit list of pre-authorized vs confirm-first actions |
| Chat or research agents that answer from training knowledge | The vendor's search trigger line (Fable 5.1, Sonnet 5.5, Haiku 5.5 — `models/claude.md`) — not a blanket "always search" |
| Creative / advisory whole-agent domain | Creative-domain kernel (`references/patterns.md`); out-of-band advice on effort |
| Non-English prompt / target output | `references/multilingual.md`: style anchor in the target language; **whitelist** of allowed English terms; thinking-language vs output-language split; final-position language gate |
| Small local model task prompt | `references/patterns.md` § small-model task prompt + `techniques-small.md`: 4-5 few-shot examples at END; Forbidden block of 3-5 named anti-patterns; EN system unlock on 2-3B; per-model override file where failure modes diverge; the model's chat-template thinking flag (not prose) |
| Per-model system override (`suites/<name>/system_<model>.md`) | Family-specific block above the Forbidden (anti-substitution for Qwen, markdown tolerance for Ministral, fold-into-first-user-turn for Gemma) — `models/small-local.md` § <family> |

When risk is real but you're unsure whether the user mitigates it elsewhere (parent CLAUDE.md, hooks, etc.), list it under Assumptions instead of `[ADD]`.

A good gap analysis is usually 0–3 `[ADD]` items, not 10.

#### Assumptions / questions

Lines whose purpose you can't confidently infer but suspect may be load-bearing. Potential additions you considered but aren't sure about. State your guess, ask for confirmation, do NOT include in Changes until confirmed.

#### Changes

Either focused `Edit` (before/after) for small fixes and confirmed additions, or a rewrite of the problematic section. Keep untouched parts untouched. One finding, one change.

### Step 6 — Explain the WHY for every change

Frontier models reason about instructions; they don't just pattern-match. The user should understand why each change helps so they can apply the principle themselves next time. One short sentence per change. Tie it to the behavior it fixes. Cite the matrix or principle when applicable.

---

## Patterns — install-once structures (full text in `references/patterns.md`)

| Pattern | Apply when | Core move |
|---|---|---|
| **Hint + literal anchor** | A deliberately hints-and-vibes prompt (named lenses, philosophy, atmosphere) targets a literal model — every current Claude, GPT-5.x / 6.x | Keep the hint AND add an anchor: a trigger clause, one concrete example, permission to surface taste-uncertainty, philosophy at the file edges. Never strip the hints |
| **Creative-domain kernel** | The agent's *whole* function is creative / advisory / narrative (game design, content, brand, coaching, brainstorm), and it feels flat, won't push back, converges on the obvious | Ask about effort first (out-of-band); install 3–6 kernel blocks from `techniques.md §25` plus a style anchor. On the Opus 5 lineage install selectively — expansion license and tone re-frame fight its defaults |
| **Small-model task prompt** | Class 2 target, structured output, single-pass compile-time task, prompt has abstract principles or no examples | Skeleton from `techniques-small.md`: neutral task line → output → whitelist → 3-5 step procedure → Forbidden (3-5 named anti-patterns) → examples at END; strip persona, principles, long blacklists, same-model self-verify, thinking-on for 2-3B |

**Prime directive interaction:** lines that look like an earlier kernel or anchor attempt are scar tissue — keep them, add only what addresses a gap.

---

## Output format for reviews

```
## Summary
[1-2 sentences: what this is, target model if relevant, overall verdict]

## Findings (existing text)
- [CRITICAL] <issue> — <line or phrase>
  Why: <reason — cite matrix.md / matrix-small.md row × column, or principle if applicable>
- [IMPROVE] <issue> — <line or phrase>
- [POLISH] <issue> — <line or phrase>

## Gap analysis (missing elements)
- [ADD] <what to add> — <failure mode prevented> — <source citation>
  Snippet to drop in:
  > <exact phrasing from techniques.md / techniques-small.md / principles.md, adapted>

Omit Gap analysis section if the prompt is genuinely complete. Prefer 0-3 items with strong justification over 10 "could help" items.

## Assumptions / questions
[Lines whose purpose you can't confidently infer, OR potential additions where you need context. State your guess, ask for confirmation, do NOT include in Changes until confirmed.]

## Changes
[Before/after snippets OR direct Edit calls, each with a one-line WHY citing matrix.md / matrix-small.md / principles.md / antipatterns-small.md.]

## Model-specific notes (only if relevant)
[Things that matter specifically for the target model — e.g. Opus 5.5 / Haiku 5.5 / GPT-6.1 Sol / Gemini 4 Argon / Gemma 4 e2b / Qwen 3.5 2B / Ministral 3B.]
```

If the user explicitly asked for a rewrite rather than a review, produce the rewritten text directly with a short rationale at the end.

---

## When NOT to trigger

- The text is regular code, marketing copy, documentation for humans, or commit messages — not instruction text for an agent
- The user asks about API parameters, SDK code, or `config.toml` settings — handoff to `claude-api` / OpenAI docs / `update-config`
- The user asks about MCP server registration, hook scripts, or permission management — handoff to `update-config`
- The user asks where to put a file or how filenames map to discovery — that's a layout question, not wording

If a request straddles wording and configuration, handle the wording part here and hand off the configuration part.

---

## Language

**The skill adapts to the user's language. Always.**

Detect the language from:
1. The user's most recent message
2. The prompt being reviewed (if no recent message — e.g., autotrigger on file open)

If the user writes in Russian, produce the **entire review in Russian** — Summary, Findings, Gap analysis, Assumptions, Changes, Model-specific notes, Contradiction-detection question. The structure, section names, and severity tags stay in English (`[CRITICAL]` / `[IMPROVE]` / `[POLISH]` / `[ADD]`) because they're audit anchors, but everything else is in the user's language.

Same rule for any other language — Spanish prompt → Spanish review; German prompt → German review.

**Mixed cases:**
- User writes in Russian, prompt itself is in English → respond in Russian, quote English snippets verbatim.
- User writes in English, prompt is in Russian → respond in English, but quote Russian snippets verbatim.
- Code identifiers (`function_name`, `field_label`) stay as-is regardless.

**Inside SKILL.md / subagent descriptions** that the user writes — keep English keywords alongside the user's language, because Claude Code, Codex, and Gemini CLI's triggering heuristics match English trigger phrases more reliably. This applies to the artifact being reviewed, NOT to your review output.

**Contradiction-detection question** in Step 4 has an English template — translate it to the user's language at runtime, keeping structure, model names, and quoted snippets verbatim.
