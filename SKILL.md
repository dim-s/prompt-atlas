---
name: prompt-atlas
description: Review and improve the WORDING of agent-facing or task-facing prompts. Covers frontier models — Claude, GPT-5.x, Gemini 3.x, Kimi, GLM, frontier Qwen, DeepSeek, Grok, Mistral frontier — and small local 2-9B models (Gemma, small Qwen, Ministral, Phi, Llama, RU tunes). Trigger when reviewing CLAUDE.md / AGENTS.md / GEMINI.md / SKILL.md / subagents / slash-commands / vendor overrides (AGENTS.deepseek.md etc.) / `suites/<name>/system*.md`. Russian — "проверь мой промпт", "улучши", "адаптируй под <модель>", "почему модель игнорирует / подменяет / не думает". English — "review my prompt", "tune for <model>", "why subagent isn't triggering", "Claude Code router prompt". Focuses on word choice, framing, structure, trigger heuristics — NOT on API params, SDK code, or file layout.
---

# Prompt Tuner

**Methodology:** apply the matrix-citation method — every finding cites a model × axis cell from `references/matrix.md`, so every recommendation is auditable and the user can challenge it from data, not authority.

Review and improve **how agent-facing prompts are written** — the words, framing, and structure that steer the model's behavior.

A complete review has two halves:
1. **What's in the text and written poorly** (Findings)
2. **What's absent but should be there** given the prompt's purpose (Gap analysis)

The second half — gap analysis against documented techniques and known failure modes — is often the higher-impact one.

This skill deals with text wording, NOT with API parameters, code, or where files live. (We surface non-wording levers like "raise `effort` before rewriting the prompt" when relevant.)

## Coverage

Two target classes — the workflow branches on which one applies (see Step 2b).

### Class 1 — Frontier agent-facing prompts

**Claude / Anthropic side:** CLAUDE.md, AGENTS.md, SKILL.md, `.claude/agents/*.md`, `.claude/commands/*.md`.

**OpenAI Codex side:** AGENTS.md, AGENTS.override.md, `.codex/skills/`, `.codex/agents/`, `.codex/commands/`, ad-hoc `codex exec` prompts.

**Google Gemini side:** GEMINI.md, AGENTS.md (when Gemini CLI is configured to read it), `.gemini/skills/`, `.gemini/agents/`, `.gemini/commands/`, headless Gemini CLI prompts.

**Moonshot Kimi side:** AGENTS.md / CLAUDE.md when accessed through Kimi Code CLI or routers; system prompts passed via `platform.moonshot.ai` API or `extra_body.thinking` toggle; agent-swarm decomposition prompts.

**Z.ai GLM side:** AGENTS.md / CLAUDE.md routed through Claude Code Router, OpenCode, Cline, Kilo Code, Cursor, or OpenClaw to GLM-5.1 / GLM-5 / GLM-4.6; direct Z.ai API system prompts. Vendor-specific overrides like `AGENTS.glm.md` if the user maintains them.

**Alibaba Qwen frontier side:** system prompts for Qwen3.7-Max / 3.6 Plus / 3.6 Max-Preview accessed via DashScope (OpenAI- or Anthropic-compatible surface), Qwen Chat web UI, or DashScope-routed agentic harnesses. **Frontier Qwen only** — small Qwen 2-9B variants belong to Class 2.

**DeepSeek side:** system + user prompts for DeepSeek V4-Pro / V4-Flash / V3.2 accessed via DeepSeek's hosted API, self-hosted endpoints, or cross-tool routers. DeepSeek's unusual user-prompt-priority makes vendor-specific overrides (`AGENTS.deepseek.md`) more common than for other vendors.

### Class 2 — Small local model task prompts

**Small-model side:** system prompts targeting 2-9B local models (Gemma 3/4, Qwen 3.5, Ministral, Phi-4-mini, Llama 3.2, and fine-tunes). Typical locations:
- `suites/<name>/system.md` and `suites/<name>/system_<model>.md` (eval / regression harnesses)
- `prompts/<topic>/<hypothesis>.md` (prompt iteration scratchpads)
- LM Studio / llama.cpp / Ollama / vLLM system prompts passed at inference time
- Inline `--system` strings in CLI prompt-harness tools

These are usually **task-facing** (compile-time: extract / classify / NL→DSL / route / score) with structured output, not agent-facing. Different axes apply — see `references/matrix-small.md`.

### Out of scope (both classes)

`temperature` / `max_tokens` / `reasoning_effort` / `thinking_level` / `extra_body.thinking` / `thinking: "off"|"high"|"max"` value tuning, vendor `config.toml` / `settings.json` configuration, MCP server registration, hook scripts, SDK code, LoRA / SFT training data design (`prompt`/`completion` JSONL pair formulation). Those belong to `claude-api`, OpenAI's docs, Google's docs, Moonshot / Z.ai / DashScope / DeepSeek vendor docs, `update-config`, or project-specific LoRA documentation.

### Reasoning-depth knobs — surface, do NOT embed (all vendors)

Each frontier vendor exposes reasoning depth as a **runtime parameter**, set out-of-band:

| Vendor | Parameter | Where set |
|---|---|---|
| Claude (Opus / Sonnet / Haiku) | `effort` (low/medium/high/xhigh) | CLI slash command (`/effort xhigh`), API param |
| OpenAI GPT-5.x | `reasoning_effort` (none/low/medium/high/xhigh) | Codex `config.toml`, API param |
| Google Gemini 3.x | `thinking_level` (minimal/low/medium/high) | Gemini CLI flags / settings, API param |
| Moonshot Kimi K2.6 | `extra_body={'thinking': {'type': 'enabled'\|'disabled'}}` | API extra_body |
| Z.ai GLM-5.x | thinking enabled at endpoint by default | endpoint default; router-injected directives are the practical lever under heavy host prompts |
| Alibaba Qwen3.7-Max | thinking mode toggle | DashScope API param / Qwen Chat UI toggle |
| DeepSeek V4 | `thinking: "off" \| "high" \| "max"` | API param |

The model **cannot change its own reasoning depth from prose**. Lines like `Use effort: xhigh for this task`, `Think at high level`, `Set thinking_level to medium for brainstorm`, `Set thinking: "max" for deep tasks`, pasted into CLAUDE.md / AGENTS.md / GEMINI.md / SKILL.md / subagent body / slash-command body are **inert text** — same failure mode as the "think step by step" antipattern (`antipatterns.md` #31).

When raising effort would help (creative agent, hints-and-vibes prompt, "think step by step" used as a substitute, kernel install on Opus 4.7, GLM running under a heavy router prompt), **mention it as a conversation point to the user** — never propose it as an `[ADD]` snippet to paste into the artifact:

> ✅ "If you're running this on Claude Code, try `/effort xhigh` before we rewrite anything — that often restores the behavior you want and costs nothing."
> ✅ "If you're on DeepSeek V4, try setting `thinking: 'max'` at the API level before tuning the prompt — the lever is more powerful than wording changes here."
> ❌ `[ADD]` to CLAUDE.md: `Use effort: xhigh for kernel work, medium for brainstorm.`

**Two exceptions where a knob legitimately lives in declarative metadata:**
1. **Codex subagent YAML frontmatter.** `.codex/agents/*.md` carries `model_reasoning_effort:` as a config knob in YAML (not prose body). Edit only when the user explicitly asks.
2. **GLM "reasoning re-injection" markers under heavy host prompts.** When the user is running GLM through Claude Code Router / OpenCode / Cline and the model isn't engaging thinking, an explicit `<reasoning_content>...</reasoning_content>` instruction in the prompt body is a legitimate workaround — not a knob substitute, but the actual recommended mitigation (see `models/glm.md` § Family-wide rules #1). Distinct from "think step by step" prose because it targets a specific documented model behavior, not a generic exhortation.

## When to auto-trigger

Trigger proactively whenever the user is working with any of the artifact types above, even without an explicit "review my prompt" request. Typical cues:

- They opened, edited, or pasted content of one of the file types
- They asked to write one from scratch
- They report that a subagent or skill "isn't triggering" or "triggers too often"
- They ask whether the wording "looks right" or "is clear enough"
- They migrated to a newer Claude or GPT-5.x version (especially 5.4 → 5.5) and want prompts updated
- They mention switching agentic environment (Claude Code ↔ Codex CLI) and need prompts adapted
- They say any variant of: "проверь", "улучши", "review", "check", "audit", "почему не работает", "make this better", "адаптируй под кодекс", "tune for GPT-5.5"
- They describe a creative / narrative / advisory / coaching / brainstorm agent and report it feels flat, mechanical, won't push back, or lost warmth on Opus 4.7 — apply the **Pattern: creative-domain kernel for Opus 4.7** below
- They report code-switching / anglicism / language-mixing symptoms in non-English output ("много англицизмов", "соскальзывает на английский", "теряет регистр", "model keeps mixing languages", "vocabulary leakage") — load `references/multilingual.md`
- They mention Kimi / GLM / Qwen frontier / DeepSeek by name, or vendor host names (Moonshot, Z.ai, Zhipu, DashScope, Alibaba Cloud, deepseek.com), or version strings (K2.6, GLM-5.1, Qwen3.7-Max, DeepSeek V4) — load `references/models/kimi.md` / `glm.md` / `qwen-frontier.md` / `deepseek.md` as relevant
- They report a vendor-specific failure that maps to a documented anti-pattern:
  - "GLM не думает / не reasoning'ит в Claude Code", "GLM rarely thinks under Claude Code's prompt", "GLM identity confusion / claims it's Claude" → `models/glm.md` § Family-wide rules #1, #3
  - "Qwen скипает разделы", "Qwen skipped a section without explicit emphasis", "Qwen needs more granular spec" → `models/qwen-frontier.md` § Family-wide rules #2, #1
  - "DeepSeek игнорирует system prompt", "DeepSeek ignored my system prompt rules", "DeepSeek 400 error on second turn / reasoning_content" → `models/deepseek.md` § Family-wide rules #1, #3
  - "Kimi swarm не вызывается", "Kimi Agent Swarms not triggering" → `models/kimi.md` § K2.6 — Agent Swarms protocol
- They mention cross-tool routers — Claude Code Router, OpenCode, Cline, Kilo Code, OpenClaw — and ask about prompt behavior; the most common case is GLM-routed setups → load `models/glm.md` § The Claude-Code-router pattern

- They are editing or writing a `suites/<name>/system.md` or per-model override `system_<model>.md`, or any prompt clearly aimed at a local 2-9B model (Gemma 3/4, Qwen 3.5, Ministral, Phi-4-mini, Llama 3.2, saiga, T-lite, Hermes, HORROR-Imatrix, etc.) — load `references/matrix-small.md` and the relevant section of `references/models/small-local.md`
- They report a small-model failure symptom: "модель подменяет / выдумывает / игнорирует правила", "не держит negation", "ломается на ripe X", "thinking leaks into output", "markdown fences keep leaking", "качество просело когда добавил правила", "EN system дал прирост" — load `references/antipatterns-small.md`
- They ask "почему этот промпт работает на Opus, но не работает на gemma / qwen / ministral" — load both matrix.md AND matrix-small.md, treat as a cross-target adaptation task

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
- Markdown body in `.claude/commands/` or `.codex/commands/` → **slash-command prompt**
- Free text the user wants to paste into a chat → **ad-hoc prompt**

If unclear, ask: "Is this a CLAUDE.md / AGENTS.md / a subagent / a skill / a slash command / something else?"

### Step 2 — Identify the target class, vendor, and model

#### Step 2a — Target class (frontier agent-facing vs small local task-facing)

This is the first split because it decides which reference matrix applies. Get it right before the vendor question.

**Frontier agent-facing** (Class 1) signals:
- File lives under `.claude/`, `.codex/`, `.gemini/`, or is a project-root `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `AGENTS.override.md`
- Frontmatter mentions Opus / Sonnet / Haiku / GPT-5.x / Gemini 3.x / 2.5
- The prompt steers a multi-turn, tool-using, agent persona (subagent body, slash command, kernel install)
- The user mentions Claude Code / Codex CLI / Gemini CLI
- Goal is hidden-intent extrapolation, taste, judgement, generalization

→ Workflow continues to Step 2b (Vendor) → Step 2c (Model version) → Step 3's frontier reference path

**Small local task-facing** (Class 2) signals:
- File lives under `suites/<name>/system.md` or `suites/<name>/system_<model>.md`
- Path mentions LM Studio (`~/.lmstudio/`), llama.cpp, Ollama, vLLM, MLX
- The user names a 2-9B local model: Gemma 3/4 (including e2b/e4b), Qwen 3.5, Ministral / Mistral 3B / 7B, Phi-4-mini, Llama 3.2, fine-tunes (saiga, T-lite, Hermes, HORROR-Imatrix, TrevorJS, …)
- Prompt is a single-pass compile-time task: extract / classify / NL→DSL / route / score / decide
- Heavy use of structured output (JSON object, JSON schema, GBNF grammar)
- The user reports a behavioral failure typical for small models: substitution bias, dropped adjectives, skipped negation, thinking-token leak, markdown emission, repetition loops

→ Workflow jumps over the frontier Step 2b/2c and goes to Step 3's small-local reference path

**Mixed target** signals:
- One prompt expected to run on both classes ("same DSL prompt for Claude AND for gemma-4-e2b")
- Migration question ("we want to move this from Opus to a local Qwen")
- Hardware tier decision ("which class will give better accuracy for our task")

→ Load BOTH reference paths; compare; flag axes where the recommendation diverges (especially `principles-tolerance`, `few-shot-density`, `position-of-critical`, `thinking-on`, `persona`, `system-language`)

**If unclear**, ask one short question. Default phrasing (translate to user's language per § Language):

> "Is this prompt for a frontier model (Claude / GPT-5.x / Gemini 3) running in an agentic CLI, or for a small local model (2-9B, like Gemma / Qwen / Ministral / Phi / Llama) running on LM Studio / llama.cpp / Ollama?"

When target class is `small local`, **skip Steps 2b/2c entirely** (vendor and model version below are frontier-specific) and proceed to Step 3 with the small-local reference path.

#### Step 2b — Vendor (frontier only)

Infer from signals; only ask if conflicting.

**Claude / Anthropic signals:** path under `.claude/`, filename `CLAUDE.md`, frontmatter with `model: opus | sonnet | haiku`, mentions of Claude / Opus / Sonnet / Haiku / Anthropic / Claude Code.

**OpenAI Codex signals:** path under `.codex/`, filename `AGENTS.override.md`, mentions of Codex / GPT / OpenAI / `codex exec` / `model_reasoning_effort` / `phase` / `text.verbosity` / `json_schema`, frontmatter with `model: gpt-5.x`.

**Google Gemini signals:** path under `.gemini/`, filename `GEMINI.md`, mentions of Gemini / Gemini CLI / `thinking_level` / `thinking_budget` / `response_json_schema` / `google-genai` / Google AI Studio / Vertex AI, frontmatter with `model: gemini-3.x`.

**Moonshot Kimi signals:** mentions of Kimi / Kimi K2 / K2.5 / K2.6 / Moonshot / `platform.moonshot.ai` / `extra_body.thinking` / `kimi-k2-thinking` / Agent Swarms / MoonViT / Kimi Code CLI; frontmatter with `model: kimi-*`. Vendor name shifts: Moonshot AI (company), Kimi (product/model).

**Z.ai GLM signals:** mentions of GLM / GLM-5.1 / GLM-5 / GLM-4.6 / Z.ai / Zhipu / `zai-org` / `api.z.ai` / `chat.z.ai` / Claude Code Router / OpenCode / Cline routing to GLM; frontmatter with `model: glm-*`. Vendor name shifts: Zhipu AI → Z.ai (rebranded late 2025 / early 2026, same company).

**Alibaba Qwen frontier signals:** mentions of Qwen3.7-Max / 3.7 Plus / 3.6 Plus / 3.6 Max-Preview / Qwen3-Max-Thinking / DashScope / `dashscope-intl.aliyuncs.com` / Alibaba Cloud Model Studio / Qwen Chat. **Frontier Qwen only** — small Qwen 2-9B variants are Class 2 and route to small-local.md.

**DeepSeek signals:** mentions of DeepSeek / V4-Pro / V4-Flash / V3.2 / V3.2-Speciale / R1 / DSA (Sparse Attention) / `reasoning_content` / `thinking: "off"/"high"/"max"` / CO-STAR / DSA Method / `deepseek-chat` / `deepseek-reasoner` API aliases.

**xAI Grok signals:** mentions of Grok / Grok 4.3 / Grok 4.20 / Grok 5 / xAI / Colossus / `grok-4.3` / Grok Skills / `x.ai` API.

**Mistral frontier signals:** mentions of Mistral Large 3 / Mistral Small 4 / Ministral 3-14B (or 8B reasoning) / Mistral AI / La Plateforme / `mistral.ai` API. **Disambiguate Ministral 3-3B / small Ministral 7B-era → Class 2 small-local.**

**Meta Muse Spark signals:** mentions of Muse Spark / Meta Superintelligence Labs / MSL / `meta.ai` private preview. Limited documentation available — treat axes as `?` until more is published.

**Cross-vendor (`AGENTS.md`):** project-root `AGENTS.md` with no vendor-specific sibling directories (no `.codex/` / `.claude/` / `.gemini/`) and no vendor-specific overrides (`AGENTS.openai.md` / `AGENTS.glm.md` / `AGENTS.deepseek.md`) is genuinely cross-vendor — tune for the lowest-common-denominator constraint set. **Default to three-vendor (Claude + GPT-5.x + Gemini)** unless the user names one of Kimi / GLM / Qwen / DeepSeek; only escalate to 4+ vendor universal when there's clear evidence (mixed-router setup, explicit user statement). See `models/_universal.md` § Cross-vendor (4+) for the warning about the 4+ trap.

#### Step 2c — Model version (frontier only)

Claude options: **Fable 5** (frontier tier above Opus, Jun 2026) / **Opus 4.8** / **Opus 4.7** / **Sonnet 4.6** / **Haiku 4.5** / **Universal Claude** / older.

OpenAI options: **GPT-5.5** (frontier) / **GPT-5.4** / **GPT-5.3 / 5.3-codex** / **GPT-5.2** / **GPT-5.1** (legacy) / **Universal GPT-5.x**.

Gemini options: **Gemini 3.1 Pro** (frontier) / **Gemini 3 Flash** / **Gemini 3.1 Flash-Lite** / **Universal Gemini 3.x** / 2.5 (legacy).

Kimi options: **Kimi K2.6** (current frontier, Apr 2026) / **K2.5** (predecessor) / **K2** (legacy, retires May 25, 2026) / **Universal Kimi**.

GLM options: **GLM-5.1** (current frontier, Apr 2026) / **GLM-5** (Feb 2026) / **GLM-4.6** (legacy, Sep 2025) / **Universal GLM**.

Frontier Qwen options: **Qwen3.7-Max** (current frontier, May 2026) / **Qwen3.7 Plus** / **Qwen3.6 Plus** / **Qwen3.6 Max-Preview** / **Qwen3-Max-Thinking** (Jan 2026 predecessor) / **Universal frontier Qwen**.

DeepSeek options: **DeepSeek V4-Pro** (current frontier coder/reasoner, Apr 2026) / **V4-Flash** (cost-efficient sibling) / **V3.2 / V3.2-Speciale** (legacy, Feb 2026) / **R1** (reasoning-only legacy) / **Universal DeepSeek**.

xAI Grok options: **Grok 4.3** (current cost-efficient flagship, Apr-May 2026) / Grok 4.20 (predecessor) / Grok 5 (Q2 2026 target, not yet shipped — don't tune for it).

Mistral frontier options: **Mistral Large 3** (Dec 2025 flagship, ecosystem still evolving) / Mistral Small 4 (Mar 2026) / Ministral 3-14B reasoning / Ministral 3-8B instruct. Ministral 3-3B → Class 2.

Meta options: **Muse Spark** (Apr 2026, closed-weight, limited docs) — treat as experimental coverage; most axes `?`.

**Recent May–June 2026 updates worth flagging in reviews:**
- **Claude Fable 5** (Jun 9) — new tier above Opus (first public Mythos-line model, $10/$50 per MTok). Three review-relevant deltas: (1) "show / explain your reasoning" instructions trigger the `reasoning_extraction` refusal classifier — audit skills when migrating; (2) subagent default **flips** vs Opus 4.8/4.7 — delegates readily, write boundaries not encouragement; (3) over-prescriptive skills from prior models can degrade output — trim. See `models/claude.md § Claude Fable 5`.
- **Gemini 3.5 Flash** (May 19) — `thinking_budget` retired → `thinking_level` enum with **default dropped from `high` to `medium`**. Silent regression risk on naive migrations from `gemini-3-flash-preview`.
- **GPT-5.5 Instant** (May 5) — new low-latency variant of GPT-5.5, same family rules apply.
- **Grok 4.3 API rollout** (Apr 30 - May 4) — 8 legacy Grok models retire **May 15, 2026**; check pinned model strings.
- **Kimi K2** (preceding K2.6) — retired **May 25, 2026**.

**Cross-vendor universal:** must behave well across two or three frontier vendors. The strictest case — opposite defaults across vendors must be reconciled. Read `models/_universal.md` § Cross-vendor (three-vendor) for Claude+GPT+Gemini; § Cross-vendor (4+) for anything broader.

If you can't infer, ask one short question. **Render in the user's detected language** (see § Language below):

> "Which model is this prompt for — Claude (Fable 5 / Opus 4.8 / 4.7 / Sonnet 4.6 / Haiku 4.5), OpenAI GPT-5.x in Codex CLI (5.3 / 5.4 / 5.5), Google Gemini 3.x (Pro / Flash / Flash-Lite), Moonshot Kimi K2.6, Z.ai GLM-5.1 / 5 / 4.6, Alibaba Qwen3.7-Max / 3.6, DeepSeek V4-Pro / Flash, or cross-vendor?"

When the answer is one of Kimi / GLM / Qwen / DeepSeek, the workflow stays on **Path A (frontier)** but loads the vendor-specific model file (`models/kimi.md` / `models/glm.md` / `models/qwen-frontier.md` / `models/deepseek.md`) — see Step 3.

### Step 3 — Read the relevant references

**The references are the source of truth.** They get updated as vendors publish new guidance and as we observe new failure modes. If you generate a review from memory, you lock the skill's quality to whatever was in the model's training data months ago.

The reference set differs by target class (decided in Step 2a). Load the path that applies, plus `principles.md` which is shared.

#### Required on every review (both classes)

1. **`references/principles.md`** — universal principles that apply across all vendors, models, classes, and systems. Read this first.

#### Path A — Frontier agent-facing references (Class 1)

Load these when target is Claude / GPT-5.x / Gemini 3:

2. **`references/matrix.md`** — frontier model × axis. Find the row(s) for the target model(s). When proposing a change, **cite the matrix cell** that drove it.

3. **`references/artifacts.md`** — find the section matching the artifact type from Step 1. For Codex artifacts, also read the matching "Codex variant" section.

4. **`references/antipatterns.md`** — scan once; match findings to documented patterns rather than improvising.

5. **Vendor-specific model file** — for nuance the matrix can't capture in a cell:
   - Claude target → `references/models/claude.md`
   - OpenAI/Codex target → `references/models/gpt.md`
   - Google Gemini target → `references/models/gemini.md`
   - Moonshot Kimi target → `references/models/kimi.md`
   - Z.ai GLM target → `references/models/glm.md`
   - Alibaba frontier Qwen target → `references/models/qwen-frontier.md` (NOT `small-local.md`, which is for 2-9B Qwen)
   - DeepSeek target → `references/models/deepseek.md`
   - xAI Grok target → `references/models/grok.md`
   - Mistral frontier target → `references/models/mistral-frontier.md` (Large 3, Small 4, Ministral 3-8B+ reasoning; small Ministral → `small-local.md`)
   - Meta Muse Spark target → no dedicated file yet (limited public docs); use matrix Table A row + flag axes as `?` until more documentation available
   - Universal Claude / GPT / Gemini / Kimi / GLM / Qwen / DeepSeek / Grok / Mistral / Cross-vendor → `references/models/_universal.md` (multiple sections — load the relevant Universal-X section + the Cross-vendor section if the prompt is multi-vendor)

6. **`references/agentic-systems/<system>.md`** — for system-specific overlays:
   - Claude Code → `references/agentic-systems/claude-code.md`
   - Codex CLI → `references/agentic-systems/codex.md`
   - Gemini CLI → `references/agentic-systems/gemini-cli.md`
   - Cross-tool `AGENTS.md` semantics → `references/agentic-systems/_common.md`
   - **Kimi Code / Z.ai routers / DashScope / DeepSeek hosted**: no dedicated agentic-systems file yet. Use `_common.md` for `AGENTS.md` cross-tool semantics; rely on the vendor model file (`models/kimi.md` / `models/glm.md` / etc.) for vendor-specific harness quirks (e.g., GLM's Claude-Code-router interference pattern is documented in `models/glm.md`).

7. **`references/techniques.md`** (load on demand) — exact wording of frontier-style snippets (uncertainty permission, verbosity control, safety/reversibility, action vs suggestion, outcome-first, creative kernel). Don't paraphrase from memory — copy the snippet and adapt.

#### Path B — Small local task-facing references (Class 2)

Load these when target is a 2-9B local model:

2. **`references/matrix-small.md`** — small-model × axis. Has its own axes (few-shot-density, position-of-critical, principles-tolerance, implicit-negation, EN-system unlock, thinking-impact, tools-impact, multi-pass-tolerance, substitution-bias, markdown-tolerance). Cite cells the same way as frontier matrix.

3. **`references/models/small-local.md`** — vendor family deltas (Gemma 3/4, Mistral / Ministral, Qwen 3.5, Phi-4-mini, Llama 3.2, RU tunes saiga / T-lite, fine-tunes Hermes / HORROR-Imatrix). Read the section for the target family. Don't skip — family-level quirks (Gemma no-system-role, Ministral markdown-emission, Qwen substitution-bias) override matrix-cell defaults.

4. **`references/techniques-small.md`** — ready-to-paste snippets for the small-model task class. Skeleton layout, few-shot-at-end template, named anti-pattern blocks, markdown-tolerant output, EN system unlock, 2-pass decompose, worker+verifier, compile-once-decompose, per-model override file layout, structured output adaptation per provider, thinking-off across providers. Don't paraphrase — copy and adapt.

5. **`references/antipatterns-small.md`** — scan once. Twelve anti-patterns specific to small models: abstract principles mid-prompt, thinking-on for 2-3B, same-model self-verify, native tool calling without few-shot, long blacklists, same-language system on 2-3B (counterintuitive — EN unlocks), frontier-style hidden-intent prose, persona block, "be concise / careful" instructions, system role on Gemma, `temperature=0` for all calls, long context with rules in middle. Each tagged with reproduced regression size.

#### Path C — Mixed target (both)

Load both Path A and Path B fully. Use the `principles.md` shared layer for everything that's vendor-agnostic. Highlight axes where Path A and Path B recommendations directly conflict (see Step 4 — Detect contradictions, with the cross-class additions).

#### Always-available

- **`references/multilingual.md`** — load when the prompt or expected output is non-English, OR when the user reports symptoms like "англицизмы", "code-switching", "model keeps switching to English", "теряет регистр", "vocabulary leakage". Applies to BOTH classes. Covers: latent-bias mechanism, model-by-model picture for 2026, Russian↔English asymmetry, style-anchor / whitelist / thinking-output split techniques with research citations, antipatterns. For small local models specifically, also see `antipatterns-small.md § 6` (same-language system) and `techniques-small.md § EN system unlocks`.

### Step 4 — Detect contradictions before findings

Before producing any findings, scan the prompt for axes where the recommendation **flips depending on which target model the user has**. Contradictions come in two flavors — within-vendor (Class 1 internal) and cross-class (Class 1 vs Class 2).

#### Within-vendor — three notorious axes (Claude / GPT / Gemini targets)

1. **Persona / "You are X"** — Gemini 3 wants it (+5%), GPT-5.5 hurts, Claude neutral, Kimi helps (Moonshot guide), GLM functional-OK-but-no-identity-pinning, frontier Qwen OK, DeepSeek brief-only.
2. **Temperature mention in body** — Gemini 3 forbids tuning (must stay 1.0), Claude/GPT-5.x are tunable, Kimi mode-split (1.0 thinking / 0.6 instant), others tunable but vendor handles.
3. **CoT scaffolding ("think step by step")** — hurts Gemini 3 and GPT-5.5, tolerated on Claude, inert on Kimi/GLM/Qwen/DeepSeek (the lever is the parameter, not prose).

Less common but possible:
- Few-shot examples (helps Claude / Kimi, hurts GPT-5.5 reasoning, mixed on Gemini, helps frontier Qwen for format)
- Negative-constraint position (Gemini drops early ones, others tolerate anywhere)
- Aggressive emphasis (Claude 4.5+ overtriggers, others mostly inert)
- XML+Markdown mixing (Gemini strict, others tolerate)

#### Cross-vendor new axes (Kimi / GLM / Qwen / DeepSeek-specific)

When the prompt has — or might be retargeted to — Kimi, GLM, Qwen, or DeepSeek, these axes can flip direction in ways the original Claude/GPT/Gemini-trained intuition doesn't predict:

| Axis | Direction by vendor | Recommendation |
|---|---|---|
| **Instruction placement (system vs user prompt)** | **DeepSeek V4: user-prompt priority** — opposite to every other vendor. System overuse → 85% of V4 errors | If target may include DeepSeek: move bulk to user prompt, keep system one-line role |
| **System prompt size sensitivity** | **GLM: heavy host system prompts suppress thinking** — Claude Code's prompt actively reduces GLM reasoning quality | If GLM-routed: target <4 KiB load-bearing; consider explicit reasoning re-injection |
| **Self-verification prompts** | **Qwen: load-bearing for completeness** — free on other vendors | If target includes Qwen: always include "review your response before finishing" |
| **Numbered requirements vs prose list** | **Qwen: skips un-emphasized sections in prose lists** — number explicitly | If target includes Qwen: convert "address the following considerations" prose to "address each of: A, B, C, D" |
| **Identity pinning ("You are GLM-5.1")** | **GLM: distillation artifact — model occasionally claims "I am Claude"**; identity-check verifications fail randomly | Strip identity-pinning from cross-vendor prompts; functional roles only |
| **XML context structure with `relevance` attrs** | **DeepSeek: 92% vs 45% accuracy** — load-bearing | Helps DeepSeek strongly; safe on other vendors |
| **JSON demanded in prose AND `response_format` set** | **DeepSeek: both required** — `response_format` alone unreliable | Demand JSON in user prompt even with API param set |
| **Granular constraints (hex colors, sizes, scopes)** | **Qwen: vagueness hurts more than other vendors** | When target includes Qwen, restore constraint detail you might strip for GPT-5.5 |
| **Agent Swarms / parallel sub-agent decomposition** | **Kimi K2.6 only** — opt-in via explicit ask; other vendors don't have this capability | Don't write swarm prompts for cross-vendor portability; vendor-specific if used |
| **Persistent reasoning across turns (preserve_thinking, keep:'all')** | **Kimi explicit, Qwen 3.6 Max-Preview explicit, DeepSeek implicit via reasoning_content round-trip** | Cross-vendor: don't write prompts that depend on visible-reasoning continuity across turns |
| **`reasoning_content` round-trip in multi-turn** | **DeepSeek V4 mandatory, V3 forbidden** — client-library concern | Don't reference in prose; flag if migrating V3→V4 prompts |

#### Cross-class — frontier vs small local

Apply when the user wants one prompt to work on both classes, or is migrating between classes. These axes go in **opposite directions** on Class 1 vs Class 2:

| Axis | Frontier (Class 1) | Small local (Class 2) |
|---|---|---|
| Abstract principles in prompt body | tolerated, often helpful | regression (−7 to −15pp on 2-3B) — must convert to few-shot demonstrations |
| Thinking / CoT on | helps (Opus 4.7 with effort=xhigh) | regression (−25 to −44pp on 2-3B) — must disable across provider params |
| Hint-and-vibes prose (creative kernel) | helps on Opus 4.7 creative work | regression — small models can't extrapolate from vibes; need literal demonstrations |
| Same-model self-verify | sometimes helps (validation pass) | regression (−40pp) — confirmation anti-bias |
| Persona block "You are an expert at X" | helps Gemini 3, neutral Claude, hurts GPT-5.5 | mostly noise on 2-3B compile tasks |
| System role | first-class on all three frontier vendors | none on Gemma family (no system role); must fold into first user turn |
| Native tool calling | works first-class | regression on 2-3B without 3-5 few-shot tool-call examples (Microsoft Phi-4-mini card admits hallucinations) |
| Long blacklists of forbidden behaviors | tolerated | regression — Pink Elephant effect; switch to whitelist + 3-5 named anti-patterns |
| Same-language system on non-EN task | preferred (matches input language) | **inverted** — EN system unlocks +4-8pp on RU/DE input for 2-3B base models |
| Critical rules position | tolerated anywhere (frontier holds attention through long context) | must be at END after Forbidden — Google's Gemma docs explicit, observed across families |

**If the prompt contains content on a contradicting axis AND the user hasn't disambiguated the target, pause and ask** before producing findings. Never silently apply a recommendation whose direction depends on something the user hasn't told us — the cost of one clarification turn is far less than the cost of a wrong recommendation that breaks a working prompt.

#### Contradiction question template

**Render in the user's detected language** (see § Language below). Template structure (English example — translate to the user's language, keep the structure, keep model names and quoted snippets verbatim):

> ⚠️ **Before continuing, I need to clarify.** The prompt contains elements where the recommendation **depends on the target model**, and you haven't specified one:
>
> 1. **Persona block** ("You are a senior engineer..."):
>    - On Gemini 3.x: **keep — +5% reasoning boost** (Google research)
>    - On GPT-5.5: **strip — hurts performance** (OpenAI guidance)
>    - On Claude: neutral
>    - On Kimi K2.6: **keep — Moonshot guide explicit**
>    - On GLM-5.1: **functional only, no identity pinning** (distillation artifact)
>    - On DeepSeek V4: **brief only — system overuse causes 85% of errors**
>
> Which model is this prompt for? If multiple, I can apply the strictest cross-vendor compromise (in `models/_universal.md` § Cross-vendor 3-way or 4+).

For specifically DeepSeek-suspicious prompts (large system prompt + complex instructions), add this contradiction:

> 2. **Instruction placement** (bulk in system prompt vs user prompt):
>    - On Claude / GPT / Gemini / Kimi / GLM / Qwen: **bulk in system** preferred
>    - On DeepSeek V4: **bulk in user prompt** — opposite default; system overuse causes 85% of errors
>
> If DeepSeek is in scope, this contradiction is the largest in the file.

#### When NOT to ask

- The vendor was already identified in Step 2 unambiguously and there's no within-vendor contradiction
- The contradiction would change a `[POLISH]` finding only — not worth interrupting the flow
- The user explicitly said "I don't care, give me the universal compromise" — apply cross-vendor wording
- The artifact is clearly tied to one vendor (file under `.claude/`, `.codex/`, `.gemini/`, or vendor override file like `AGENTS.deepseek.md`, `AGENTS.glm.md`) — no contradiction to resolve

#### After clarification

Once the user answers, **proceed with Step 5 (findings)**. Note in the review which findings were contingent on the clarification — gives auditability.

If the user picks "cross-vendor / universal" — apply the strictest compromise from `models/_universal.md` § Cross-vendor and cite the table.

---

### Step 5 — Produce findings, gap analysis, then changes

#### Findings — what's broken in existing text

Tag each:
- `[CRITICAL]` — will cause the prompt to misfire (vague subagent description that never triggers, negative instructions with no alternative, missing verification on a side-effect command)
- `[IMPROVE]` — will meaningfully improve behavior (missing WHY, no examples, negative framing, verbose rule list)
- `[POLISH]` — small wording cleanup

Each finding points to the exact line or phrase. For each finding, note current behavior vs. expected behavior after the change. **When citing the matrix, name the row × column** that drove the finding — makes the recommendation auditable.

#### Gap analysis — what's missing

Findings catch what's broken; gap analysis catches what was never written. Often higher-impact.

Method:

1. **State the prompt's purpose in one sentence** — what behavior is it trying to steer?
2. **Enumerate plausible failure modes** for that purpose. Pick the ones THIS prompt would plausibly hit.
3. **Cross-reference references** for snippets directly addressing those failure modes — `principles.md`, `matrix.md`, `agentic-systems/<system>.md`, `techniques.md`.
4. **Propose each missing element as `[ADD]`** with: (a) failure mode prevented, (b) source citation, (c) exact snippet to drop in. Don't paraphrase — copy and adapt.

**Common high-value additions by artifact type** (always justify from THIS prompt, not this list):

| Artifact | Often missing |
|---|---|
| Subagents with tool access | Verification step (`agentic-systems/claude-code.md` § Verification); permission to express uncertainty (`principles.md` #11); output format with priority tags |
| Subagents meant for delegation | "Use proactively" / "Use when..." in description; negative scope boundary against sibling agent |
| CLAUDE.md / AGENTS.md | WHY for non-obvious rules; concrete commands instead of abstractions; load-bearing invariants at edges; negative-scope section ("don't touch X") |
| SKILL.md descriptions | Explicit trigger clauses with file names / user phrasings; "Do NOT use when..." boundary; bilingual keywords if domain is non-English |
| Slash commands with side effects | Verification step (`agentic-systems/claude-code.md`); explicit reversibility wording; cleanup-after-yourself |
| Slash commands meant for skill behavior | `disable-model-invocation: true` if side effects (Claude Code) |
| Ad-hoc prompts | Functional role sentence; success criteria; one or two examples (Claude) or zero-shot + criteria (GPT-5.5) |
| Long-context / RAG prompts | Document-at-top / question-at-bottom; quote-grounding; permission to say "I don't know" |
| Agentic coding prompts | Anti-overengineering snippet; anti-hard-coding; subagent-orchestration guidance |
| Per-task creative work on Opus 4.7 | Universal creativity unlockers (`techniques.md §25` top); pair with project-specific anchors. Surface non-prompt lever: raise `effort` before rewriting prompts |
| Whole-agent creative domain on Opus 4.7 (game design, narrative, content, brand, coaching, brainstorm, advisory) | Install **creative-domain kernel** (`techniques.md §25` bottom + Pattern section in this file): 3–6 composable blocks at system-prompt level + style anchor. **Out-of-band advice to the user (NOT pasted into the file):** which `effort` to run — `high` default, `medium` for pure brainstorm, `xhigh` only for structured creative. Detection cues in Pattern section below |
| Non-English prompt / target output (Russian, German, Japanese, etc.) | Load `references/multilingual.md`. Common adds: style anchor in target language; **whitelist** of allowed English terms (NOT blacklist); separate thinking-language from output-language for reasoning tasks; final-position language gate (`<output_language>...`); system-prompt language matches target where the agent is permanently localized |
| Small local model task prompt (`suites/<name>/system.md`, system prompt for Gemma / Qwen / Ministral / Phi / Llama 2-9B) | Load `references/matrix-small.md`, `references/techniques-small.md`, `references/antipatterns-small.md`, and `references/models/small-local.md` for the target family. Common adds: 4-5 few-shot examples at END mirroring failure modes; Forbidden block of 3-5 named anti-patterns (`techniques-small.md § Anti-defaults block`); EN system unlock for non-EN inputs on 2-3B; per-model override file when one base has divergent failure modes across models. Common strips: abstract principles in body; persona block; native tool calling without few-shot; thinking-on for 2-3B; same-model self-verify |
| Per-model system override (`suites/<name>/system_<model>.md`) | Family-specific block above the Forbidden: anti-substitution block for Qwen, markdown-tolerance for Mistral / Ministral, "fold into first user turn" for Gemma. Cite `models/small-local.md § <family>` for the exact recommended block |
| Prompt targeting Moonshot Kimi K2.6 | If swarm work expected but not framed: add explicit "Decompose this into parallel sub-tasks..." line. Otherwise, often complete — Kimi tolerates Claude-style prompts well. Watch for stripped persona / few-shot from a GPT-5.5 port (restore them). Cite `models/kimi.md § Family-wide rules` |
| Prompt targeting Z.ai GLM-5.1 routed through Claude Code / OpenCode / Cline | (1) Reasoning re-injection block (`<reasoning_content>` markers + "write detailed reasoning before answering") if heavy host system prompt suppresses thinking. (2) Strip identity-pinning if present. (3) Trim total load-bearing AGENTS.md size to <4 KiB. Cite `models/glm.md § The Claude-Code-router pattern` |
| Prompt targeting Alibaba Qwen3.7-Max | (1) Self-verification instruction ("before finishing, review your response..."). (2) Numbered requirements where critical sections must not be skipped. (3) Restore granular constraints (hex colors, sizes, scopes) if stripped during a GPT-5.5 port. Cite `models/qwen-frontier.md § Family-wide rules #2-5` |
| Prompt targeting DeepSeek V4-Pro / V4-Flash | (1) Move bulk of instruction from system prompt to user prompt; keep system as one-line functional role. (2) Add XML-tagged `<context relevance="...">` structure if input has multiple sections. (3) Demand JSON in user prompt prose even when `response_format` is set. Cite `models/deepseek.md § Family-wide rules #1, #2, #6` |

When risk is real but you're unsure whether the user mitigates it elsewhere (parent CLAUDE.md, hooks, etc.), list it under Assumptions instead of `[ADD]`.

A good gap analysis is usually 0–3 `[ADD]` items, not 10.

#### Assumptions / questions

Lines whose purpose you can't confidently infer but suspect may be load-bearing. Potential additions you considered but aren't sure about. State your guess, ask for confirmation, do NOT include in Changes until confirmed.

#### Changes

Either focused `Edit` (before/after) for small fixes and confirmed additions, or a rewrite of the problematic section. Keep untouched parts untouched. One finding, one change.

### Step 6 — Explain the WHY for every change

Claude models reason about instructions; they don't just pattern-match. The user should understand why each change helps so they can apply the principle themselves next time. One short sentence per change. Tie it to the behavior it fixes. Cite the matrix or principle when applicable.

---

## Pattern: hint + literal anchor

Some prompts are intentionally hints-and-vibes style — named lenses ("apply KISS / reuse-first"), philosophical framing, atmospheric description. On older Claude (pre-4.7) the model extrapolated from vibes automatically; on Opus 4.7 and GPT-5.5 it doesn't.

The anti-pattern is to react by stripping hints. That kills the frame the user was relying on for generalization, tone, and taste. The correct move is to **keep the hint AND add a literal anchor** — preserving extrapolation *and* giving the literal model a foothold.

Four anchors to pair with load-bearing hints:

- **Trigger clause** — *when* to invoke + *what observable output*. "Перед значимой правкой кода прогоняй через эти линзы; если линза подсвечивает — озвучь одной фразой до кода."
- **One concrete example** showing the hint applied to a real decision.
- **Permission to surface taste-uncertainty**: *"Если не уверен — предложи списком, жди выбора. Не выбирай молча."*
- **Place philosophy at file edges** (top or bottom), not middle.

Apply this pattern when target is a literal model (Opus 4.7 / GPT-5.5) AND the prompt has deliberate hints-and-vibes framing. Skip if the prompt is already all-literal-rules.

**Non-prompt lever:** on Claude Code with Opus 4.7, raising `effort` to `xhigh` often restores hint-sensitivity better than any prompt edit. Surface this as a conversation point before proposing a rewrite — do not paste effort guidance into the file (effort is a CLI / config / API parameter, see § *Effort / reasoning_effort / thinking_level — surface, do NOT embed*).

Full snippet examples in `references/principles.md`.

---

## Pattern: creative-domain kernel for Opus 4.7

A close cousin of hint+anchor, but for a different problem. When the agent's **entire function** is creative / advisory / narrative / non-technical (game design, narrative, content, brand, strategy, coaching, brainstorming partner, design crit, fiction editor), Opus 4.7's defaults work against it on every turn: literalism kills inference, the flatter tone reads as cold, first-obvious-answer convergence kills divergent options, format-drift pushes prose into bullets.

The anti-pattern is to address this per-task — adding unlockers every turn. The correct move on a permanent-creative agent is to install a **kernel** at the system-prompt level (CLAUDE.md role section, subagent body, role-system text) that overrides Opus 4.7 defaults once.

#### Detection — when to apply

Triggers (at least two should match):
- Agent's primary output is ideas / options / critique / narrative / pitch / scenario
- Stated domain: game design, narrative, screenwriting, content, brand, marketing, strategy, coaching, therapy-adjacent, brainstorm, advisory, design
- User symptoms: "feels mechanical", "doesn't push back", "won't suggest things I didn't ask", "lost the warmth", "gives one obvious answer", "skips to bullets when I want prose"
- Russian cues: «сухо», «скучно», «без огонька», «не додумывает», «слишком технично», «как QA, не как дизайнер»

When prompt-atlas detects this case, the review should:
1. **First surface the non-prompt lever — ask the user about their effort setting, do NOT embed a recommendation in the file.** `low`/`medium` defeats any kernel. Advise out-of-band: for creative work default to `high`; `medium` for pure brainstorm; `xhigh` only when the creative task has a structured frame; `low` is almost always wrong here. Effort lives in the CLI / config / API call, not in CLAUDE.md or the kernel body — see § *Effort / reasoning_effort / thinking_level — surface, do NOT embed* above.
2. **Then propose installing the kernel** — 3–6 composable blocks from `techniques.md §25 "Whole-agent creative kernel"`, NOT all of them. Pick by domain.
3. **Always include the style anchor recommendation** — ask the user to paste 1–3 paragraphs in target voice. This is the single highest-leverage line on 4.7 for creative work.

#### Composable kernel blocks (full snippets in `techniques.md §25`)

Each line overrides a documented 4.7 default. Pick by which default is hurting the user:

| 4.7 default that conflicts | Kernel block to install |
|---|---|
| Obedient executor — literal interpretation | Role re-frame ("senior collaborator, not executor") |
| No inference / loss of "додумывание" | Expansion license ("treat request as minimum scope, raise adjacent moves") |
| Flatter, less validation-forward tone | Tone re-frame ("warm, direct, prose by default, no bullets/emoji/summary") |
| Checklist-clarify drift on ambiguity | Ambiguity handling ("interpret most likely meaning and proceed") |
| First-obvious-answer convergence | Default divergence ("3+ options from different roots, include a risky one") |
| Validation-forward agreement | Default pushback ("steelman opposition in one sentence before agreeing") |
| Habitual scaffolding ("I'd be happy to" etc.) | Anti-defaults block (named prohibitions) |

#### When NOT to apply

- Agent is mostly technical (coding, refactoring, ops) and only occasionally creative → use per-task unlockers (`techniques.md §25` top half) instead. The kernel's tone block will pollute regular technical output.
- Cross-vendor universal prompt — the tone block is Claude-shaped (warmth needs explicit reinstall specifically on 4.7); Gemini 3 and GPT-5.5 react differently. Keep role re-frame + expansion license + divergence; drop tone.
- Sonnet 4.6 / Haiku 4.5 targets — Sonnet less literal than 4.7 so lighter touch works; Haiku needs more concrete examples and fewer abstract clauses.
- Agent has strong existing register that conflicts (formal legal advisor, compliance bot) — adapt tone, keep the rest.

**Prime directive interaction**: if the existing prompt already has lines that look like a kernel attempt — keep them, even if awkwardly worded. They're scar tissue from observed failures. Only add blocks that address gaps; don't rewrite an existing tone line unless the user asks.

---

## Pattern: small-model task prompt (Class 2)

Small local models on compile-time tasks live in a completely different prompt-engineering regime than frontier creative agents. This pattern is the small-model analog of the creative kernel — a standard layout that overrides the model's failure modes once at the structural level.

#### Detection — when to apply

All four should match:
- Target is a 2-9B local model (Gemma 3/4, Qwen 3.5, Ministral, Phi-4-mini, Llama 3.2, or fine-tune)
- Output is structured (JSON object, JSON schema, DSL command, classification label) — not free-form prose
- Task is single-pass compile-time (extract / classify / NL→DSL / route / decide), not multi-turn conversation
- The prompt currently has abstract principles, hint-and-vibes prose, or no examples

#### What to install

The full skeleton lives in `techniques-small.md § Skeleton: small-model task prompt`. The non-negotiable structural elements:

1. **Task identity** in one neutral sentence at top (no persona block, no "You are an expert at...")
2. **Output section** before whitelist — shape, "no prose, no fence, no explanation"
3. **Whitelist / vocabulary** in a table
4. **Resolution procedure** in 3-5 numbered steps
5. **Forbidden** — 3-5 named anti-patterns, each one concrete failure mode
6. **Examples** at END — 4 for Gemma, 5 for Qwen, 4-5 for Ministral, mirroring failure classes
7. **Trailing "now read input below"** to prime the model

#### Family-specific adaptations

After installing the skeleton, layer on family-specific blocks from `models/small-local.md`:
- **Gemma**: no system role — fold instructions into first user turn; EN system on non-EN task
- **Mistral / Ministral**: markdown-tolerant output paragraph + named anti-patterns block ("CRITICAL: rules are sacred...")
- **Qwen**: anti-substitution block with 5 examples; always pass `enable_thinking=False`
- **Phi-4-mini**: avoid native tool calling without few-shot tool-call examples
- **Llama 3.2 / Hermes**: standard system role; JSON tool calling template

#### What NOT to install (small-model anti-patterns to strip)

Pre-existing prompt has any of these → flag as `[CRITICAL]` or `[IMPROVE]`:
- Abstract principles in body ("be careful with negations", "every adjective is mandatory")
- Hint-and-vibes prose copied from a frontier agent prompt
- Persona block ("You are an expert X with N years of experience")
- Long blacklist (>5 prohibitions, especially if no positive example follows)
- Same-model self-verify pass
- Native tool calling without few-shot tool-call examples
- System prompt in non-English when input is non-English and model is a base 2-3B (EN-system unlock applies)
- Thinking on / `reasoning_effort: high` for 2-3B (cite the −25 to −44pp regressions from `antipatterns-small.md § 2`)

Full inventory of 12 anti-patterns with reproduced regression sizes in `antipatterns-small.md`. Don't just flag from memory — cite the specific anti-pattern number so the user can audit.

#### When NOT to apply this pattern

- Target is a frontier model (Class 1) — use the frontier workflow path instead
- Target is a small local model but the task is free-form prose generation (chat companion, narrative continuation) — closer to a creative-kernel adaptation than a compile-time skeleton
- The user has a working benchmark suite showing the current prompt at >90% and is asking for marginal improvements — diminishing returns; recommend benchmark-driven iteration over more prompt edits

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
[Things that matter specifically for the target model — Opus 4.7 / Sonnet 4.6 / Haiku 4.5 / GPT-5.5 / Gemma 4 e2b / Qwen 3.5 2B / Ministral 3B / etc.]
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
