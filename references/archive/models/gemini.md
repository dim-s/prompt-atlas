# Archive — legacy sections from `models/gemini.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

## Gemini 3 Flash

Balanced model — speed and capability between Pro and Flash-Lite.

### Behaviors that shape wording

- **Same family rules apply** — concise, identity-based, no temperature tuning, no CoT
- **`thinking_level: low` is a common choice for latency-sensitive use** — if so, system instruction *"think silently"* helps reduce visible-reasoning overhead
- **Few-shot tolerance is more forgiving than Pro** — when format is genuinely strict, examples help here more than on Pro

### When to use Flash vs Pro

Flash is the default for interactive sessions, agentic loops, and high-volume work. Pro is for one-shot deep reasoning, long-context analysis, hard problems. Wording is the same; reasoning depth differs.

## Gemini 3.7 Flash (August 13, 2026 — previous Flash frontier)

`gemini-3.7-flash`, GA on release, three weeks after 3.6 Flash. Google positions it as "our most intelligent workhorse model yet for coding and agents." It is the new default model powering the **Antigravity agent** (Gemini Managed Agents / Antigravity SDK) and the engine behind **Gemini Spark** (Google AI Pro / Ultra personal agent).

### Behaviors that shape wording

- **Better instruction-following fidelity and first-pass accuracy** — Google reports stronger gains on coding / issue resolution; FrontierCode 1.1 Main **43.6% vs 34.4%** (3.6 Flash), DeepSWE v1.1 **65.3 vs 49.0**, AutomationBench **30.4 vs 17.0**
- **`thinking_level` values `low` / `medium` / `high`, default `medium`** — same default level as 3.5 Flash, not `high`; re-check prompts that assumed 3.6-era defaults (default level is a silent-regression risk on naive ports, same class as the 3.5 Flash migration gotcha)
- **1M context, 64K max output** — long-context patterns apply unchanged
- Family rules unchanged: concise input, identity-based persona, no CoT scaffolding, negatives at the end, one structure format, sampling params still deprecated
- Intro price **$0.75 / $3.75 per MTok — half of 3.6 Flash's original rate** — through 31.12.2026, then $1.50 / $7.50; Google applied the same new rate to 3.6 Flash

### When 3.7 Flash specific tuning helps

- Agentic coding / agent loops migrating off 3.6 Flash — the first-pass / instruction-fidelity gains are the pitch; re-check any scaffold added to stop 3.6-era thrashing
- Web-dev and knowledge-dense work (GDP.pdf 34.0 vs 22.0), enterprise workflow automation (AutomationBench 30.4 vs 17.0)

## Gemini 3.5 Flash Cyber (July 21, 2026 — closed access)

A 3.5 Flash fine-tune for finding and fixing security vulnerabilities, announced alongside the two models above but **not generally available** — access is limited to governments and trusted partners through CodeMender. Noted here so a future coverage pass doesn't re-investigate it: there is nothing to tune against, and a prompt targeting it can't be validated.

## Gemini 3.5 Flash (May 19, 2026 — previous Flash frontier)

A larger jump than the version number suggests. Beats Gemini 3.1 Pro on **coding and agentic benchmarks** (Terminal-Bench 2.1 76.2%, MCP Atlas 83.6%, GDPval-AA 1656 Elo) while 3.1 Pro still wins **abstract reasoning** (Humanity's Last Exam +4.2 pts, ARC-AGI-2 +5).

### Critical migration gotcha — `thinking_level` default dropped

The integer `thinking_budget` from Gemini 3 Flash Preview is **retired**. New `thinking_level` enum: `minimal` / `low` / `medium` (**default**) / `high`.

**Silent regression risk:** the default moved from `high` (on the Preview) to `medium`. A naive port `gemini-3-flash-preview` → `gemini-3.5-flash` reasons less than the old prompt did, with no error. Audit prompts that depended on the prior default; set `thinking_level: high` explicitly if the workload needs it.

**Also retuned:** `low` is no longer "skip reasoning" — Google retuned it specifically for code/agentic tasks. Use `low` deliberately for agentic loops where you want fast turns; don't avoid it as if it disables thinking.

### Wording differences from 3 Flash

- Family rules unchanged (concise, identity-based, no temperature tuning, no CoT scaffolding)
- Coding benchmarks now Flash-leading — older Flash prompts that punted hard coding to Pro can be revisited
- Long-context (1M) behavior same; documents-top / question-bottom anchor still applies

### When 3.5 Flash specific tuning helps

- Agentic coding loops migrating from 3.1 Pro for cost/speed
- High-volume agentic workloads where 3.5 Flash now matches Pro on coding-shaped tasks
- Migration audits from any `gemini-3-flash-preview` codebase

## Gemini 3.1 Flash-Lite (March 2026)

Speed/cost-optimized — 363 tokens/sec, $0.25/$1.50 per 1M tokens, 1M context. Designed for high-volume, latency-sensitive workloads.

### Behaviors that shape wording

- **Concise wording is critical** — every token costs latency at this throughput
- **`thinking_level: low` is typical** — explicit "think silently" in system instruction is common
- **Few-shot for format helps more than on Pro** — Lite needs more guidance for strict formats
- **Bounded scope wins** — like Haiku 4.5, Lite excels at classification, extraction, summarization, translation. If you find yourself writing a 10-step workflow for Lite, promote to Flash or Pro
- **Tool descriptions matter more** — push tool guidance into descriptions, keep system prompt minimal

### As a subagent

Lite is the right default when the subagent's job is bounded and read-only: classifiers, extractors, cheap pre-filters, language tasks. Use Flash or Pro for the main conversation where reasoning quality matters.

## Gemini 2.5 Pro / Flash (legacy — for migration reviews)

Older generation. When migrating from 2.5 to 3.x, the changes are large enough that Google recommends a fresh-baseline approach:

### Wording fixes when migrating 2.5 → 3.x

| Old 2.5-style | 3.x-style |
|---|---|
| "Think step by step." / "First, do X. Then do Y." | (Delete.) Set `thinking_level: high` if reasoning is shallow. |
| Long block of "DO NOT do X / Y / Z" | Replace with positive scoped instructions ("Use the provided context for deductions"). Move remaining negatives to the END of the prompt. |
| Few-shot block of 5 examples | Test zero-shot first; keep examples only if format drifts. |
| Mixed XML + Markdown structure | Pick one. |
| Persona absent or weak | **Add identity-based persona** ("You are a planner...") — measurable boost. |
| `temperature: 0.3` for deterministic output | **Remove.** Gemini 3 default 1.0 is required. |
| `thinking_budget: <number>` | Migrate to `thinking_level: minimal/low/medium/high`. |
