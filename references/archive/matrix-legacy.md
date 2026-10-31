# Archive — legacy rows from `matrix.md`

Moved here on 2026-10-31 (v2.0.0). Rows for retired, deprecated or two-plus-generations-old models, kept verbatim as they stood — **not maintained**. Load only for a prompt that still targets one of these models or for a migration off one. Column meanings are the same as in `../matrix.md`.

## Table A

| Model | Literalism | Generalizes scope | Persona / "act as" | Few-shot for format | Few-shot for reasoning | Aggressive emphasis | Step-by-step prescription |
|---|---|---|---|---|---|---|---|
| **Opus 4.8** (legacy) | very high | no — must state scope | OK if functional, 1 line | helps (3-5) | helps if relevant | overtriggers — reserve for safety | tolerated; sometimes appreciated |
| **Opus 4.7** (legacy) | very high | no — must state scope | OK if functional, 1 line | helps (3-5) | helps if relevant | overtriggers — reserve for safety | tolerated; sometimes appreciated |
| **Sonnet 4.6** (previous) | high | rarely | OK | helps | helps | overtriggers | tolerated |
| **Opus 4.6 (legacy)** | medium | yes — extrapolates | helps strongly | helps | helps | works as intended | tolerated |
| **GPT-5.5** | high | partial | hurts — strip persona | hurts on reasoning | hurts | mostly inert noise | **treats as noise** — outcome-first |
| **GPT-5.4** | medium-high | yes | mild help | helps | mixed | inert | tolerated |
| **GPT-5.3 / 5.3-codex** | medium | yes | OK | helps (3-5) | helps | inert | tolerated |
| **GPT-5.2** | medium | yes | helps (CTCO era) | helps | helps | inert | encouraged (CTCO) |
| **GPT-5.1** | low-medium | yes — needs scaffolding | helps | helps | helps | inert | needed |
| **Gemini 3 Flash** | high | partial | helps (identity-based) | mixed | mixed | inert | CoT not needed |
| **Gemini 3.1 Flash-Lite** | high (concise wins for speed) | partial | helps (cheaper persona use) | helps when format strict | mixed | inert | CoT not needed |
| **Gemini 2.5 Pro / Flash (legacy)** | medium | yes | helps strongly | helps | helps | works | encouraged (CoT scaffolding common) |
| **GLM-5.1** (Z.ai, Apr 2026 — previous) | medium-high | partial | OK, but **distillation identity confusion** — model occasionally claims "I am Claude" | helps | helps | inert | tolerated; helps for long-horizon |
| **GLM-5 / 4.6 (legacy)** | medium | yes | OK | helps | helps | inert | tolerated |
| **Qwen3.6 Plus / Max-Preview (legacy)** | high | partial | OK | helps | helps | inert | tolerated |
| **DeepSeek V4-Flash** (Apr 2026) | high | partial | OK in brief system | helps | helps | inert | needed for complex tasks |
| **DeepSeek V3.2 (legacy)** | medium-high | partial | helps | helps | helps | inert | tolerated |
| **GPT-5.5 Instant** (May 5 2026) | high (same family as 5.5) | partial | hurts — same family rule | hurts on reasoning | hurts | inert | treats as noise — outcome-first |
| **Gemini 3.5 Flash** (May 19 2026) | high — concise dominates | partial | helps (identity-based, family-consistent) | mixed | mixed | inert | **CoT not needed** — but `thinking_level` default dropped to `medium` (was `high`); naive port silently reasons less |
| **Grok 4.3** (xAI, Apr 30 / May 4 2026) | medium-high | yes — generalizes well | OK; persona helps for tone but no strong directional preference documented | helps when format strict | helps | inert | tolerated; built-in reasoning makes scaffolding unnecessary |
| **Gemini 3.7 Flash** (Aug 13 2026 — previous Flash) | high — concise dominates | partial | helps (identity-based, family-consistent) | mixed | mixed | inert | CoT not needed; **`thinking_level` default medium** — set explicitly if depth needed; intro price half of 3.6 Flash |
| **Grok 4.5** (xAI, Jul 2026 — previous recommended) | medium-high (4.3 family behavior; **no 4.5 prompting guide published**) | yes | OK; no directional preference documented | helps when format strict | helps | inert | tolerated; built-in reasoning makes scaffolding unnecessary; **`reasoning_effort` (default high) exists since 4.5** |
| **Haiku 4.5** | medium-high | rarely | OK | helps; needed when format strict | mixed | overtriggers | needed for complex tasks |

## Table B

| Model | Reasoning depth lever | Default tone | Output format preference | Verbosity control |
|---|---|---|---|---|
| **Opus 4.8** (legacy) | `effort` (low/medium/high/xhigh) — primary; **default `high` on all surfaces incl Claude Code** | direct, less validation, fewer emojis | prose constraints OK; XML tags for multi-part | calibrates to task; explicit if forced |
| **Opus 4.7** (legacy) | `effort` (low/medium/high/xhigh) — primary | direct, less validation, fewer emojis | prose constraints OK; XML tags for multi-part | calibrates to task; explicit if forced |
| **Sonnet 4.6** (previous) | `effort`; thinking off by default | direct, similar to Opus 4.8/4.7 | prose / XML | calibrates |
| **GPT-5.5** | `reasoning_effort` (none/low/medium/high/xhigh) — primary | efficient, task-oriented, no padding | **`json_schema` strongly preferred** | `text.verbosity` parameter |
| **GPT-5.4** | `reasoning_effort` (default `none`) | direct | `json_schema` preferred | `text.verbosity` |
| **GPT-5.3 / 5.3-codex** | `reasoning_effort` | direct (codex more action-biased) | `json_schema` preferred | `text.verbosity` |
| **GPT-5.2** | `reasoning_effort` | direct | CTCO + prose acceptable | `text.verbosity` |
| **GPT-5.1** | `reasoning_effort` | varies | prose | parameter or prose |
| **Gemini 3 Flash** | `thinking_level` (often `low` for latency + "think silently" in system instruction) | direct | `json_schema` preferred | implicit |
| **Gemini 3.1 Flash-Lite** | `thinking_level: low` typical (speed-optimized) | direct, terse | `json_schema` preferred | implicit |
| **GLM-5.1** (previous) | thinking **enabled by default at the /chat/completions endpoint** — model decides whether to engage chain-of-thought | direct | `json_schema` (recommended) / `json_object` (legacy) | implicit |
| **GLM-5 / 4.6 (legacy)** | thinking on by default | direct | `json_schema` / `json_object` | implicit |
| **DeepSeek V3.2 (legacy)** | reasoning enabled separately (R1-style); tool calls and reasoning were mutually exclusive | direct | `json_object`; demand JSON in prose | implicit |
| **GPT-5.5 Instant** | `reasoning_effort` — same family knob, tuned for low-latency default | direct, terse | `json_schema` preferred | `text.verbosity` |
| **Gemini 3.5 Flash** | `thinking_level` (minimal/low/medium-**default**/high) — **default dropped from `high` to `medium`** vs prior 3-Flash-Preview; "low" retuned for code/agentic tasks | direct, efficient | `response_json_schema` preferred | implicit |
| **Grok 4.3** | built-in reasoning (no separate toggle documented); model decides reasoning depth autonomously | direct, terse, action-biased | structured output via OpenAI-compatible API | implicit |
| **Gemini 3.7 Flash** (previous) | `thinking_level` (low/medium/**default**/high) — default `medium`; set explicitly for depth-critical work | direct, efficient; higher first-pass accuracy (FrontierCode 43.6 vs 34.4 on 3.6) | `response_json_schema` preferred | implicit |
| **Haiku 4.5** | `effort` (limited range) | direct, terse | prose; explicit format needed | terse default |

## Table C

| Model | Tool guidance location | Subagent default | "Use proactively" needed? | MCP tool description critical? |
|---|---|---|---|---|
| **Opus 4.8** (legacy) | system prompt OK | spawns fewer — explicit ask needed | improved — triggers required tools reliably; favors reasoning, so raise effort / instruct for *more* tool use | yes |
| **Opus 4.7** (legacy) | system prompt OK | spawns fewer — explicit ask needed | yes — undertriggers | yes |
| **Sonnet 4.6** (previous) | system prompt OK | spawns more by default | sometimes | yes |
| **GPT-5.5** | **inside tool description** (strong split) | spawns what's defined | yes | yes |
| **GPT-5.4** | tool description preferred | mid | yes | yes |
| **GPT-5.3-codex** | tool description preferred | action-biased | yes | yes |
| **GPT-5.2** | mixed | mid | yes | yes |
| **GPT-5.1** | system prompt | mid | yes | yes |
| **Gemini 3 Flash** | system instruction OK | mid | yes | yes |
| **Gemini 3.1 Flash-Lite** | terse — push tool guidance into tool descriptions | bounded scope works best | yes | yes |
| **GLM-5.1** (previous) | system prompt + tool description; **but heavy host system prompts suppress thinking** | spawns what's defined; **8-hour autonomous runs** observed in long-horizon tasks (6,000+ tool calls) | yes | yes |
| **GLM-5 / 4.6** | mixed | mid | yes | yes |
| **DeepSeek V3.2 (legacy)** | similar; R1 lineage made reasoning + tools mutually exclusive | mid | yes | yes |
| **GPT-5.5 Instant** | inside tool description (family rule) | spawns what's defined | yes | yes |
| **Gemini 3.5 Flash** | system instruction OK; tool descriptions for tool specifics | spawns what's defined; **leads agentic tool calling** on Terminal-Bench 2.1 (76.2%) and MCP Atlas (83.6%) | yes | yes |
| **Grok 4.3** | tool description preferred | **#1 on Artificial Analysis agentic tool-calling leaderboard**; Grok Skills feature for persistent custom expertise | yes | yes |
| **Haiku 4.5** | system prompt OK | conservative — name tool explicitly | yes | yes |

## Table D

| Model | Migration style | Patches from prev version |
|---|---|---|
| **Opus 4.8** (legacy) | runs out-of-box on 4.7 prompts; `high` effort default | check effort-default latency/token cost; trim now-redundant "use the tool" nudges (under-triggering fixed); verbosity recalibrates |
| **Opus 4.7** (legacy) | forward-compatible from 4.6; tone shifts | trim "avoid AI slop" frontend blocks; trim "after every 3 tool calls summarize" |
| **Sonnet 4.6** (previous) | forward-compatible | trim "be thorough / use X when in doubt" — overtriggers |
| **GPT-5.5** | **fresh baseline required** (OpenAI explicit) | strip 5.4-era process-step prescription; strip "think step by step"; move format to `json_schema`; cut few-shot for reasoning |
| **GPT-5.4** | minor patches from 5.3 | introduce `phase` field for Responses API agents |
| **GPT-5.3 / 5.3-codex** | minor patches from 5.2 | mostly unchanged |
| **GPT-5.2** | adopted CTCO formally | introduced CTCO scaffolding |
| **GPT-5.1** | legacy | overdue for rewrite |
| **Gemini 3 Flash** | minor patches from earlier 3.x | similar — strip CoT, fix thinking knob |
| **Gemini 3.1 Flash-Lite** | new (March 2026) | fresh — no legacy patches usually |
| **Gemini 2.5 Pro / Flash → 3.x** | **CoT-stripping rewrite** | remove "think step by step" / chain-of-thought scaffolding; rely on `thinking_level: high`; review temperature uses; persona blocks become more load-bearing (will be obeyed strongly) |
| **GLM-5.1** (previous) | minor patches from GLM-5 | mostly unchanged from 5.0; review SWE-bench-tuned prompts |
| **GLM-5 / 4.6** | from GLM-4.6 → GLM-5: re-evaluate against system-engineering benchmarks | rewrite for system-engineering focus over pure coding; review tool-calling assumptions |
| **DeepSeek V3.2 (legacy)** | from R1: tool/reasoning unified; better long-context | update R1-era prompts that assumed reasoning-vs-tool tradeoff |
| **GPT-5.5 Instant** | from 5.5: same family, lower latency | same migration as 5.5 (fresh baseline); pick Instant when latency matters more than reasoning depth |
| **Gemini 3.5 Flash** | from 3 Flash Preview / 3.1 Flash | **silent regression risk**: `thinking_budget` int retired → `thinking_level` enum with **`medium` default** (was `high` on Preview). Naive port reasons less. Also audit any code reading `thinking_budget` |
| **Grok 4.3** | from Grok 4.20: 40% input price cut, 1M context, video input added | move to Grok 4.3 for cost/feature; Grok 5 still upcoming (Q2 2026) |
| **Grok 4.5** | vendor-recommended over 4.3 — **but the context window shrinks 1M → 500K** | `[CRITICAL]` if the prompt architecture assumes 1M (whole-repo dumps, full history replay): chunk, retrieve or summarize. Also note the **200K pricing step** — crossing it doubles input *and* output rates |
| **Haiku 4.5** | forward-compatible | mostly unchanged from 4.0/4.5 era |
