# Vendor / model differences — the matrix

The single source of truth for **how models differ** along behavioral axes that affect prompt wording. Each row is a model. Each column is an axis. Cells encode the model's bias along that axis.

When reviewing a prompt:
1. Identify the target model(s).
2. Read the relevant rows here.
3. Drill into `models/<vendor>.md` and `agentic-systems/<system>.md` ONLY for nuance the matrix can't capture.

When adding a new model: add a row here first. If a row needs more than 1-2 sentences per cell, push the detail into `models/<vendor>.md` and keep the matrix terse.

---

## How to read a cell

- **Numerical scale** (literalism, persona-tolerance, etc.): `low / medium / high` — model's bias along that axis. Higher = more of that property.
- **Recommendation scale** (few-shot, emphasis, etc.): `helps / neutral / hurts` — what adding this technique does on this model.
- **Mechanism column** (reasoning lever, output format): names the actual API knob or technique that works best.
- **Migration column**: how to port a prompt from the previous version.
- **Default column**: what the model does without explicit guidance.

When a cell says `?` — the axis hasn't been tested for this model; treat conservatively (see Universal-prompt rules in `principles.md`).

---

## Table A — Core wording behaviors

| Model | Literalism | Generalizes scope | Persona / "act as" | Few-shot for format | Few-shot for reasoning | Aggressive emphasis | Step-by-step prescription |
|---|---|---|---|---|---|---|---|
| **Opus 4.7** | very high | no — must state scope | OK if functional, 1 line | helps (3-5) | helps if relevant | overtriggers — reserve for safety | tolerated; sometimes appreciated |
| **Sonnet 4.6** | high | rarely | OK | helps | helps | overtriggers | tolerated |
| **Haiku 4.5** | medium-high | rarely | OK | helps; needed when format strict | mixed | overtriggers | needed for complex tasks |
| **Opus 4.6 (legacy)** | medium | yes — extrapolates | helps strongly | helps | helps | works as intended | tolerated |
| **GPT-5.5** | high | partial | hurts — strip persona | hurts on reasoning | hurts | mostly inert noise | **treats as noise** — outcome-first |
| **GPT-5.4** | medium-high | yes | mild help | helps | mixed | inert | tolerated |
| **GPT-5.3 / 5.3-codex** | medium | yes | OK | helps (3-5) | helps | inert | tolerated |
| **GPT-5.2** | medium | yes | helps (CTCO era) | helps | helps | inert | encouraged (CTCO) |
| **GPT-5.1** | low-medium | yes — needs scaffolding | helps | helps | helps | inert | needed |
| **Gemini 3.1 Pro** | high — concise / over-analyzes verbose prompts | partial | **+5% boost — keep as identity** ("You are a planner") | mixed (test) | mixed (over-analyzes verbose) | inert noise | **CoT no longer needed** (use `thinking_level: high` instead) |
| **Gemini 3 Flash** | high | partial | helps (identity-based) | mixed | mixed | inert | CoT not needed |
| **Gemini 3.1 Flash-Lite** | high (concise wins for speed) | partial | helps (cheaper persona use) | helps when format strict | mixed | inert | CoT not needed |
| **Gemini 2.5 Pro / Flash (legacy)** | medium | yes | helps strongly | helps | helps | works | encouraged (CoT scaffolding common) |
| **Kimi K2.6** (Moonshot, Apr 2026) | medium-high | partial | **helps** — official docs encourage role assignment | helps (official guide explicit) | helps | inert noise | tolerated — official guide encourages explicit steps |
| **GLM-5.1** (Z.ai, Apr 2026) | medium-high | partial | OK, but **distillation identity confusion** — model occasionally claims "I am Claude" | helps | helps | inert | tolerated; helps for long-horizon |
| **GLM-5 / 4.6 (legacy)** | medium | yes | OK | helps | helps | inert | tolerated |
| **Qwen3.7-Max** (Alibaba, May 2026) | **very high** — explicit > vague more than Claude/GPT | partial | OK (closed-weight reasoning model) | helps when format strict | helps | inert | tolerated; "self-verify" prompts boost completeness |
| **Qwen3.6 Plus / Max-Preview (legacy)** | high | partial | OK | helps | helps | inert | tolerated |
| **DeepSeek V4-Pro** (Apr 2026) | high — slightly more literal than Claude | partial | OK in **brief** system; full instructions belong in user prompt | helps (place at end of first user turn) | helps | inert | tolerated |
| **DeepSeek V4-Flash** (Apr 2026) | high | partial | OK in brief system | helps | helps | inert | needed for complex tasks |
| **DeepSeek V3.2 (legacy)** | medium-high | partial | helps | helps | helps | inert | tolerated |
| **GPT-5.5 Instant** (May 5 2026) | high (same family as 5.5) | partial | hurts — same family rule | hurts on reasoning | hurts | inert | treats as noise — outcome-first |
| **Gemini 3.5 Flash** (May 19 2026) | high — concise dominates | partial | helps (identity-based, family-consistent) | mixed | mixed | inert | **CoT not needed** — but `thinking_level` default dropped to `medium` (was `high`); naive port silently reasons less |
| **Grok 4.3** (xAI, Apr 30 / May 4 2026) | medium-high | yes — generalizes well | OK; persona helps for tone but no strong directional preference documented | helps when format strict | helps | inert | tolerated; built-in reasoning makes scaffolding unnecessary |
| **Mistral Large 3** (Dec 2025, ecosystem evolving) | medium-high | yes | helps (functional persona) | helps | helps | inert | tolerated; reasoning variants exist for explicit mode |
| **Meta Muse Spark** (Apr 8 2026, closed) | ? — limited public docs | ? | ? | ? | ? | ? | ? |

**Reading guide:**
- *"hurts"* = adding this technique measurably degrades output on this model.
- *"overtriggers"* = the model applies the rule beyond intended scope; use sparingly.
- *"strip persona"* = "You are an expert at X" lines should be removed; outcome-first beats persona.
- *"+5% boost"* = Google reports identity-based prompting ("You are a planner") boosts reasoning on Gemini 3 by ~5%. **Opposite default to GPT-5.5** — flag in cross-vendor reviews.
- *"distillation identity confusion"* (GLM) = the model occasionally responds "I am Claude, created by Anthropic" — a documented training artifact. Persona blocks that hard-pin identity may misfire; functional-role wording is safer.
- *"DeepSeek user-prompt priority"* = DeepSeek's official guidance (echoed in V4 practitioner guides) puts **core instructions in the user message**, with a brief system prompt ("You are a senior architect" style). Opposite default to most other vendors; flag when porting Claude/GPT system prompts to V4.

---

## Table B — Reasoning, tone, output

| Model | Reasoning depth lever | Default tone | Output format preference | Verbosity control |
|---|---|---|---|---|
| **Opus 4.7** | `effort` (low/medium/high/xhigh) — primary | direct, less validation, fewer emojis | prose constraints OK; XML tags for multi-part | calibrates to task; explicit if forced |
| **Sonnet 4.6** | `effort` | direct, similar to Opus 4.7 | prose / XML | calibrates |
| **Haiku 4.5** | `effort` (limited range) | direct, terse | prose; explicit format needed | terse default |
| **GPT-5.5** | `reasoning_effort` (none/low/medium/high/xhigh) — primary | efficient, task-oriented, no padding | **`json_schema` strongly preferred** | `text.verbosity` parameter |
| **GPT-5.4** | `reasoning_effort` (default `none`) | direct | `json_schema` preferred | `text.verbosity` |
| **GPT-5.3 / 5.3-codex** | `reasoning_effort` | direct (codex more action-biased) | `json_schema` preferred | `text.verbosity` |
| **GPT-5.2** | `reasoning_effort` | direct | CTCO + prose acceptable | `text.verbosity` |
| **GPT-5.1** | `reasoning_effort` | varies | prose | parameter or prose |
| **Gemini 3.1 Pro** | `thinking_level` (minimal/low/medium/**high default**) — replaces legacy `thinking_budget` | direct, less verbose, efficient | `response_mime_type` + `response_json_schema`; XML or Markdown (not both) | implicit; explicit only for conversational tone |
| **Gemini 3 Flash** | `thinking_level` (often `low` for latency + "think silently" in system instruction) | direct | `json_schema` preferred | implicit |
| **Gemini 3.1 Flash-Lite** | `thinking_level: low` typical (speed-optimized) | direct, terse | `json_schema` preferred | implicit |
| **Kimi K2.6** | `extra_body={'thinking':{'type':'enabled'\|'disabled'}}` — two-mode toggle, **thinking is default** | warm, helpful, conversational by default | `json_object` / JSON Mode; XML/triple-quote delimiters helpful | implicit; specify length explicitly when needed |
| **GLM-5.1** | thinking **enabled by default at the /chat/completions endpoint** — model decides whether to engage chain-of-thought | direct | `json_schema` (recommended) / `json_object` (legacy) | implicit |
| **GLM-5 / 4.6 (legacy)** | thinking on by default | direct | `json_schema` / `json_object` | implicit |
| **Qwen3.7-Max** | thinking-mode toggle — required-on for best results, **adds 45-60s latency** | direct, **higher abstention** ("I don't know" more often than competitors) | `json_object` / structured prompt | implicit |
| **DeepSeek V4-Pro / Flash** | `thinking: "off" \| "high" \| "max"` — three-level lever; **tool calls work in thinking mode** (unlike R1) | direct, slightly more literal than Claude | `json_object` + **must still demand JSON in prose** even with `response_format` set | implicit |
| **DeepSeek V3.2 (legacy)** | reasoning enabled separately (R1-style); tool calls and reasoning were mutually exclusive | direct | `json_object`; demand JSON in prose | implicit |
| **GPT-5.5 Instant** | `reasoning_effort` — same family knob, tuned for low-latency default | direct, terse | `json_schema` preferred | `text.verbosity` |
| **Gemini 3.5 Flash** | `thinking_level` (minimal/low/medium-**default**/high) — **default dropped from `high` to `medium`** vs prior 3-Flash-Preview; "low" retuned for code/agentic tasks | direct, efficient | `response_json_schema` preferred | implicit |
| **Grok 4.3** | built-in reasoning (no separate toggle documented); model decides reasoning depth autonomously | direct, terse, action-biased | structured output via OpenAI-compatible API | implicit |
| **Mistral Large 3** | reasoning variants exposed separately (instruct vs reasoning models in the family) | direct | `json_object` standard | implicit |
| **Meta Muse Spark** | ? — limited public docs | ? | ? | ? |

**Cross-vendor rule for reasoning depth:** never write "think harder" or "think step by step" in the body. The lever is the API parameter on all current vendors (Claude `effort`, OpenAI `reasoning_effort`, Gemini `thinking_level`, Kimi `extra_body.thinking`, GLM endpoint-default, Qwen mode-toggle, DeepSeek `thinking`). Wording substitutes: "Verify against criteria...", "List edge cases first...".

**Gemini-specific gotcha:** `thinking_level` and legacy `thinking_budget` cannot coexist in one request — returns 400 error. When migrating from 2.5 prompts, audit code for both.

**Temperature gotcha (Gemini-only):** Google **strongly recommends keeping temperature at 1.0**. Lowering causes looping / degraded reasoning. **This is opposite to Claude / GPT-5.x where temperature is freely tunable.** Cross-vendor wording: don't reference temperature in the prompt body; let API config handle it per vendor.

**Kimi-specific temperature split:** Moonshot's model card recommends `temperature=1.0, top_p=1.0` for Thinking mode and `temperature=0.6, top_p=0.95` for Instant mode. Don't hardcode a single temperature into a Kimi-targeted prompt.

**DeepSeek-specific multi-turn gotcha:** the `reasoning_content` field returned by V4 **must be round-tripped** on subsequent turns or the API returns 400. V3.2 / R1 **rejected** the same field — a hard breaking change on the migration path. Wording in cross-version DeepSeek prompts should not assume either behavior; let the client library handle it.

**GLM-specific system-prompt gotcha:** Z.ai's GLM models have thinking enabled by default but **heavy system prompts (especially Claude Code's) suppress the reasoning judgment**, and the model rarely engages chain-of-thought under that load. Cross-tool routers (Claude Code Router, OpenCode) work around this by injecting custom `<reasoning_content>` markers and explicitly instructing the model to "write detailed reasoning before answering" — not by removing the host system prompt. Treat heavy persistent-context files as actively hostile to GLM thinking unless mitigations are present.

---

## Table C — Tool use, subagents, delegation

| Model | Tool guidance location | Subagent default | "Use proactively" needed? | MCP tool description critical? |
|---|---|---|---|---|
| **Opus 4.7** | system prompt OK | spawns fewer — explicit ask needed | yes — undertriggers | yes |
| **Sonnet 4.6** | system prompt OK | spawns more by default | sometimes | yes |
| **Haiku 4.5** | system prompt OK | conservative — name tool explicitly | yes | yes |
| **GPT-5.5** | **inside tool description** (strong split) | spawns what's defined | yes | yes |
| **GPT-5.4** | tool description preferred | mid | yes | yes |
| **GPT-5.3-codex** | tool description preferred | action-biased | yes | yes |
| **GPT-5.2** | mixed | mid | yes | yes |
| **GPT-5.1** | system prompt | mid | yes | yes |
| **Gemini 3.1 Pro** | system instruction OK; tool descriptions for tool specifics | spawns what's defined; "subagents" + "remote subagents" supported | yes | yes |
| **Gemini 3 Flash** | system instruction OK | mid | yes | yes |
| **Gemini 3.1 Flash-Lite** | terse — push tool guidance into tool descriptions | bounded scope works best | yes | yes |
| **Kimi K2.6** | system prompt OK; tool descriptions also work | **Agent Swarms** (300 parallel sub-agents, 4000 steps) — explicit ask needed for swarm decomposition | yes — undertriggers swarms without explicit framing | yes |
| **GLM-5.1** | system prompt + tool description; **but heavy host system prompts suppress thinking** | spawns what's defined; **8-hour autonomous runs** observed in long-horizon tasks (6,000+ tool calls) | yes | yes |
| **GLM-5 / 4.6** | mixed | mid | yes | yes |
| **Qwen3.7-Max** | system instruction + tool description (OpenAI + Anthropic API compat both supported) | **35-hour autonomous claim** (vendor-reported, unverified externally); 1,000+ tool calls per run | yes | yes |
| **DeepSeek V4-Pro / Flash** | tool description preferred; **keep system prompt brief** (system overuse → 85% error rate per V4 guides) | spawns what's defined; tool calls work in thinking mode | yes | yes |
| **DeepSeek V3.2 (legacy)** | similar; R1 lineage made reasoning + tools mutually exclusive | mid | yes | yes |
| **GPT-5.5 Instant** | inside tool description (family rule) | spawns what's defined | yes | yes |
| **Gemini 3.5 Flash** | system instruction OK; tool descriptions for tool specifics | spawns what's defined; **leads agentic tool calling** on Terminal-Bench 2.1 (76.2%) and MCP Atlas (83.6%) | yes | yes |
| **Grok 4.3** | tool description preferred | **#1 on Artificial Analysis agentic tool-calling leaderboard**; Grok Skills feature for persistent custom expertise | yes | yes |
| **Mistral Large 3** | tool description preferred | spawns what's defined | yes | yes |
| **Meta Muse Spark** | ? | designed for "multi-agent orchestration" per vendor positioning; specifics ? | ? | ? |

**Cross-vendor compromise:** put tool-specific guidance INSIDE the tool description. Costs nothing on Claude / Gemini / Kimi / Grok / Mistral, matters a lot on GPT-5.x and DeepSeek V4 (system overuse penalty).

---

## Table D — Migration / version policy

| Model | Migration style | Patches from prev version |
|---|---|---|
| **Opus 4.7** | forward-compatible from 4.6; tone shifts | trim "avoid AI slop" frontend blocks; trim "after every 3 tool calls summarize" |
| **Sonnet 4.6** | forward-compatible | trim "be thorough / use X when in doubt" — overtriggers |
| **Haiku 4.5** | forward-compatible | mostly unchanged from 4.0/4.5 era |
| **GPT-5.5** | **fresh baseline required** (OpenAI explicit) | strip 5.4-era process-step prescription; strip "think step by step"; move format to `json_schema`; cut few-shot for reasoning |
| **GPT-5.4** | minor patches from 5.3 | introduce `phase` field for Responses API agents |
| **GPT-5.3 / 5.3-codex** | minor patches from 5.2 | mostly unchanged |
| **GPT-5.2** | adopted CTCO formally | introduced CTCO scaffolding |
| **GPT-5.1** | legacy | overdue for rewrite |
| **Gemini 3.1 Pro** | from Gemini 3 Pro Preview (deprecated 2026-03-09) | strip CoT scaffolding (use `thinking_level: high`); remove explicit temperature; switch SDK from `google-generativeai` (EOL Nov 2025) to `google-genai`; audit `thinking_budget` → `thinking_level` |
| **Gemini 3 Flash** | minor patches from earlier 3.x | similar — strip CoT, fix thinking knob |
| **Gemini 3.1 Flash-Lite** | new (March 2026) | fresh — no legacy patches usually |
| **Gemini 2.5 Pro / Flash → 3.x** | **CoT-stripping rewrite** | remove "think step by step" / chain-of-thought scaffolding; rely on `thinking_level: high`; review temperature uses; persona blocks become more load-bearing (will be obeyed strongly) |
| **Kimi K2.6** | forward-compatible from K2.5; **K2 retires May 25, 2026** | trim "pure text" assumptions (K2.6 is multimodal); switch text-only image handling code; review thinking-toggle code (preserve_thinking semantics added) |
| **GLM-5.1** | minor patches from GLM-5 | mostly unchanged from 5.0; review SWE-bench-tuned prompts |
| **GLM-5 / 4.6** | from GLM-4.6 → GLM-5: re-evaluate against system-engineering benchmarks | rewrite for system-engineering focus over pure coding; review tool-calling assumptions |
| **Qwen3.7-Max** | from Qwen3.6 / 3-Max-Thinking | budget for higher abstention ("I don't know" jumps); trim factual-recall-dependent prompts; expect 1M context but validate against your task |
| **DeepSeek V4-Pro / Flash** | **API breaking from V3.2** on `reasoning_content` (V4 requires round-trip, V3 rejects) | rebuild multi-turn flows to round-trip `reasoning_content`; audit prefix cache hashing; move complex instructions from system → user prompt |
| **DeepSeek V3.2 (legacy)** | from R1: tool/reasoning unified; better long-context | update R1-era prompts that assumed reasoning-vs-tool tradeoff |
| **GPT-5.5 Instant** | from 5.5: same family, lower latency | same migration as 5.5 (fresh baseline); pick Instant when latency matters more than reasoning depth |
| **Gemini 3.5 Flash** | from 3 Flash Preview / 3.1 Flash | **silent regression risk**: `thinking_budget` int retired → `thinking_level` enum with **`medium` default** (was `high` on Preview). Naive port reasons less. Also audit any code reading `thinking_budget` |
| **Grok 4.3** | from Grok 4.20: 40% input price cut, 1M context, video input added | move to Grok 4.3 for cost/feature; Grok 5 still upcoming (Q2 2026) |
| **Mistral Large 3** | from Mistral Large 2: MoE jump; reasoning variants split out | re-evaluate reasoning vs instruct variant choice in the family |
| **Meta Muse Spark** | first-generation Meta Superintelligence Labs model | no migration path from Llama 4 — Muse Spark is closed-weight, not a Llama successor |

---

## Table E — Persistent-context behavior (CLAUDE.md / AGENTS.md)

| Model / system | Hard cap | Soft degradation | Hierarchy |
|---|---|---|---|
| **Claude Code (any Claude model)** | none | context rot beyond ~300 lines | one file per scope (root, parent dirs, child dirs on-demand); imports via `@path` |
| **Codex CLI (any GPT-5.x model)** | 32 KiB (`project_doc_max_bytes`) — silent drop past cap | rot still applies | hierarchical AGENTS.md + `.override.md` pattern; concatenated top-down |
| **Gemini CLI (any Gemini 3.x model)** | none documented (soft) | rot still applies | hierarchical `GEMINI.md` (also `AGENTS.md` via `settings.json: { context.fileName }`); `~/.gemini/GEMINI.md` global + workspace + parents — concatenated; supports `@file.md` imports |
| **Cursor** | ? (TBD when added) | ? | `.cursorrules` single file |
| **Aider** | ? | ? | `CONVENTIONS.md` |
| **Kimi Code (Moonshot)** | ? — undocumented cap | likely rot, untested at scale | follows Claude Code conventions for routed mode; native mode TBD |
| **Z.ai API + cross-tool routers (GLM)** | none on API; **router-mediated system-prompt size is the binding constraint** | thinking suppression past large prompts (documented) | use slim project-level `AGENTS.md` (<4 KiB) — heavy persistent context measurably suppresses GLM thinking |
| **DashScope (Qwen)** | ? — OpenAI-compat surface | 1M nominal but no independent long-context verification | works inside any tool that talks OpenAI API; Anthropic-compat surface also supported |
| **DeepSeek API** | ? on the chat endpoint; **`reasoning_content` round-trip mandatory** | DSA reasoning quality degrades past ~500K of 1M context | XML-tagged context structure recommended (92% vs 45% unstructured per V4 guides) |

**Cross-vendor rule:** target combined hierarchy size **under 8 KiB** for the file's load-bearing rules. Reserves 24 KiB for growth before Codex's hard cap surprises you. Gemini CLI accepts `AGENTS.md` natively when `settings.json` is configured — making `AGENTS.md` viable as a true cross-tool standard across Claude Code (via `@import`), Codex CLI (native), and Gemini CLI (native via config).

**GLM-specific cap:** even though Z.ai's API has no documented size cap, GLM's reasoning judgment is measurably suppressed by heavy host system prompts (Claude Code, OpenCode, Cline). For GLM-routed setups, keep `AGENTS.md` under ~4 KiB and prefer in-message `<reasoning_content>` cues over relying on the endpoint's default thinking mode.

---

## How to use the matrix during a review

### Step 1 — Pick rows

- Single-model prompt → 1 row
- Universal-Claude → all 3 Claude rows; intersect (write for the strictest)
- Universal-GPT-5.x → all GPT-5.x rows; intersect
- Cross-vendor → strictest cells across both vendors

### Step 2 — Pick relevant axes

- Behavior of the prompt under review tells you which axes matter:
  - Is it a long persistent-context file? → table E
  - Does it prescribe step-by-step? → table A column "Step-by-step prescription"
  - Does it use a persona ("You are...")? → table A column "Persona"
  - Does it specify output format in prose? → table B column "Output format"
  - Does it use few-shot examples? → table A columns "Few-shot..."
  - Does it use aggressive emphasis? → table A column "Aggressive emphasis"
  - Does it tell the model to "think harder"? → table B column "Reasoning depth lever"
  - Does it spawn subagents? → table C column "Subagent default"

### Step 3 — Compare the cell to what the prompt does

- If the prompt does the thing the cell calls "neutral / helps" → no finding.
- If the prompt does the thing the cell calls "hurts / overtriggers / strip / fresh baseline" → `[IMPROVE]` or `[CRITICAL]` finding.
- If the prompt skips a technique the cell says "needed" → `[ADD]` finding.

### Step 4 — Cite the matrix in the review

When proposing a change, say *which row × column* drove it. Example: *"Persona line 'You are a senior engineer' → strip on GPT-5.5 (table A, column 'Persona/act-as' = hurts)."* This makes the recommendation auditable and the user can challenge it from the matrix instead of from your authority.

---

## Adding a new model — checklist

When extending the skill to cover a new model (e.g., Gemini 2.5, DeepSeek V4, Grok 4):

- [ ] Add a row to **all five tables** (A through E). Use `?` for unknowns; `?` is honest, guesses are dangerous.
- [ ] Add a `models/<vendor>.md` file ONLY if the new model has nuance the matrix can't capture in 1-2 sentences per cell.
- [ ] If the new model runs in a new agentic system, add `agentic-systems/<system>.md` with system-specific quirks (file paths, hierarchy, hooks, headless mode).
- [ ] Update `models/_universal.md` if the new model changes the cross-vendor compromise (e.g., a third vendor with opposite defaults to both Claude and GPT).
- [ ] Don't add new principles to `principles.md` unless the new model invalidates a universal — that's a much bigger event than adding a row.

---

## Adding a new agentic system — checklist

When extending the skill to cover a new system (e.g., Cursor, Windsurf, Aider, Roo Code):

- [ ] Add `agentic-systems/<system>.md` with: persistent-context file format, file paths, frontmatter conventions, skill / agent / slash-command equivalents, hooks model, headless mode, MCP support.
- [ ] Add a row to **table E** for persistent-context cap behavior.
- [ ] Add to `agentic-systems/_common.md` if the system reads `AGENTS.md` (most do — it's becoming the cross-tool standard).
- [ ] Update SKILL.md Step 1 (artifact identification) and Step 2a (vendor identification) ONLY if the new system uses file paths the current routing doesn't recognize.
- [ ] Update SKILL.md description with English+Russian trigger phrases for the new system's filenames.
