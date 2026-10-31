# Universal / multi-model / cross-vendor wording rules

Rules for prompts that must work across multiple models or vendors. The differences themselves live in `matrix.md`. The shared principles live in `principles.md`. This file covers the **compromises** — what to write when the prompt has to be acceptable to several models with conflicting defaults.

---

## When this file applies

Multiple overlapping cases. Read the relevant section.

| Case | Section |
|---|---|
| Prompt must work across multiple Claude versions (Fable 5.1 + Opus 5.5 + Sonnet 5.5 + Haiku 5.5, and the previous 5.0 generation) | § Universal Claude → `models/claude.md` |
| Prompt must work across multiple GPT versions (GPT-6.x + 5.6) | § Universal GPT → `models/gpt.md` |
| Prompt must work across multiple Gemini models (Argon + 3.x Pro / Flash / Flash-Lite) | § Universal Gemini → `models/gemini.md` |
| Prompt must work across two or three frontier vendors (Claude + GPT + Gemini) | § Cross-vendor (three-vendor) |
| Prompt must work across multiple Kimi versions (K3.1 Pro / K3 / K2.x) | § Universal Kimi |
| Prompt must work across multiple Z.ai GLM versions | § Universal GLM |
| Prompt must work across multiple frontier Qwen versions | § Universal frontier Qwen |
| Prompt must work across DeepSeek V4-Pro + V4.1 Flash | § Universal DeepSeek |
| Prompt must work across four-or-more frontier vendors (any mix of Claude / GPT / Gemini / Kimi / GLM / Qwen / DeepSeek) | § Cross-vendor (4+) — and read the warning |

Most production CLAUDE.md / AGENTS.md fall into one of these — pure single-model prompts are rarer than they look. **4+ vendor cross-everything universal prompts are usually a trap** — see § Cross-vendor (4+) before committing to the design.

---

## Universal Claude

**Lives in `models/claude.md § Universal / multi-model prompts`** — one copy, maintained with the model sections (rules 1–9, the within-family divergence table, the checklist). The short version for a cross-vendor reader: every current Claude is literal (state scope); sampling parameters are gone (API version `2026-10-21`); forced `tool_choice` and a thinking off-switch can't be assumed; "show your reasoning" and "think less" lines are strips; self-review rituals hurt the Opus 5 lineage while Sonnet 5.5 / Haiku 5.5 ship an official *task-check* paragraph for low effort — state verification as part of the task.

---

## Universal GPT

**Lives in `models/gpt.md § Universal / multi-version GPT prompts`** (GPT-6 Astra / 6.1 Sol / 6.1 Terra / 6 Sol / 6 Luna + 5.6; older 5.x in `archive/models/gpt.md`). The short version for a cross-vendor reader: write outcome-first and state each rule once (5.6 measured the cost of repetition); length as a task requirement, `text.verbosity` for the default; output contract in `json_schema`; no persona; effort `none` and sampling exist only on some tiers (Astra / 6.1 Sol have neither), so a cross-tier prompt assumes neither.

---

## Universal Gemini

**Lives in `models/gemini.md § Universal Gemini prompts (Argon + 3.x)`** (Gemini 4 Argon, 3.1 Pro, 3.8 / 3.6 Flash, 3.5 Flash-Lite). The short version: concise, no CoT scaffolding (depth is `thinking_level`, whose ladder and default differ per model — Argon and 3.8 Flash have no `minimal`); identity persona documented for 3.x, unverified on Argon; XML or Markdown, not both; negative constraints at the end; no sampling in the body or the code (ignored on Argon / 3.6+, 400 announced); no global length disposition (3.6 Flash terser, 3.8 Flash works harder).

---

## Universal Kimi

A prompt that works across K3.1 Pro, K3, K2.7-Code and K2.6. The K2 series (K2.7-Code, K2.6) closed to new users on 31.10.2026 and retires at the end of December — a K2.x prompt in scope is due for migration to K3.1 Pro, not universalization. K2.5 and K2 are archived.

### Rules

**1. No fast non-reasoning path is universal.** Only K2.6 has the Thinking / Instant toggle; K3 / K3.1 Pro always think (depth via `reasoning_effort` low/high/max); K2.7-Code cannot stop thinking. Never write "answer immediately without reasoning".

**2. Sampling is fixed on the hosted API** (temperature 1.0 / top_p 0.95 on the K3 line and K2.7-Code; K2.6 by mode). Never put sampling in the body.

**3. Multi-turn history round-trips the complete assistant message** (`reasoning_content`, `tool_calls`) on the K3 line; dynamically loaded tool definitions stay in history.

**4. Agent Swarms are a K2.6-lineage feature** with no documented K3-line bounds — state the decomposition explicitly and don't depend on swarm scale.

**5. Functional persona and 3–5 format examples are family-safe.**

### Universal-Kimi checklist

- [ ] No "answer without reasoning" / Instant-mode assumption
- [ ] No sampling in the body; no reliance on non-default sampling in code
- [ ] Complete assistant messages round-tripped (K3 line)
- [ ] Swarm decomposition stated, not assumed
- [ ] Functional persona; standard OpenAI tools format
- [ ] No model name hardcoded

---

## Universal GLM

A prompt that works across GLM-5.3 (incl. 5.3-Flash / FlashX) and GLM-5.2. GLM-5.1 / 5 / 4.6 are archived. The family-wide rule — heavy host system prompts suppress thinking — applies to both; the mitigation is the `reasoning_effort` param (5.3: `low`/`high`/`max`, default `max`, **no off-switch**; 5.2: `high`/`max` plus the `thinking` toggle). Prose `<reasoning_content>` re-injection is the fallback for routers that don't forward the param.

### Rules

**1. Keep host-side overhead in mind.** Through Claude Code Router, OpenCode, Cline or similar, every kilobyte stacks on the router's injection — target <4 KiB load-bearing.

**2. Don't pin model identity.** The "I am Claude" distillation artifact is documented on GLM-5 / 5.1 and `?` on 5.2 / 5.3 — functional roles only either way.

**3. Reasoning lever = the param;** keep a re-injection block only where the router doesn't forward it. On 5.3 a "don't think" line is unimplementable.

**4. `json_schema` over `json_object`.** **5. Outcome-first beats step prescription.**

### Universal-GLM checklist

- [ ] System + AGENTS.md <4 KiB load-bearing (the 1M context doesn't lift this — it's a reasoning-gate ceiling)
- [ ] No identity pinning — functional roles only
- [ ] No "answer without reasoning" line
- [ ] Reasoning lever handled: `reasoning_effort` where forwarded, re-injection only as router fallback
- [ ] `json_schema` for structured output; no "think step by step"
- [ ] No model name hardcoded

---

## Universal frontier Qwen

A prompt that works across Qwen3.8-Max, Qwen3.8-Flash, Qwen3.8-Omni-Flash and Qwen3.7-Max. 3.7 Plus / 3.6 Plus / 3.6-Max-Preview / 3-Max-Thinking are archived. **Frontier Qwen only** — small local Qwen is `small-local.md § Alibaba Qwen`.

### Rules

**1. Write for granularity.** Frontier Qwen skips un-emphasized sections — number requirements and keep constraints concrete (hex colors, sizes, scopes); restore detail stripped during GPT tuning.

**2. Add self-verification** — load-bearing on Qwen for completeness ("before finishing, review your response and check that every requested element is covered"). In a set that includes the Opus 5 lineage, gate it (Cross-vendor (4+)).

**3. The thinking lever is a parameter.** 3.8: `reasoning_effort` (low/medium/xhigh, default xhigh) + `enable_thinking`; the **Max open weights (`Qwen3.8-2.4T-A95B`) are thinking-only**. 3.7-Max: the toggle. Never write "think step by step" or "don't think".

**4. Reasoning continuity differs by surface:** `preserve_thinking` is on by default on hosted Max / Omni-Flash and in the open Flash-Next card, **off** on hosted `qwen3.8-flash`. Don't write prompts that depend on earlier reasoning being visible.

**5. Don't assume 1M-context reliability** — validate before long-context patterns.

### Universal-frontier-Qwen checklist

- [ ] Constraints explicit; requirements numbered
- [ ] Self-verification present (gated if Opus 5.x is in scope)
- [ ] Reasoning via parameters, no CoT scaffolding, no "don't think"
- [ ] No dependence on cross-turn reasoning visibility
- [ ] 1M context validated, not assumed; vision support confirmed per model (3.7-Max is text-only)
- [ ] No model name hardcoded

---

## Universal DeepSeek

A prompt that works across DeepSeek V4-Pro and V4.1 Flash (`deepseek-flash`, build 1017). V4-Flash, V4-Flash-Vision-Exp, V3.2 and R1 are retired or legacy — pinned old ids are already served by 1017 (`archive/models/deepseek.md`). Wording is forward-compatible; the breaks are API-level.

### Rules

**1. Instruction placement: authoring fresh, put the bulk in the user prompt** with a brief system prompt — a V4 practitioner rule citing R1-era vendor advice, twice not reproduced on V4-Flash-0731. Porting a working system-side body: measure before re-architecting.

**2. XML-tagged context with `relevance` attrs** (~92% vs ~45% reported).

**3. Demand JSON in prose AND set `response_format`.**

**4. Don't reference `reasoning_content` in prose** — with tools it must be round-tripped (400 otherwise), without tools it is dropped; the client handles it.

**5. Depth is the effort parameter** (`low`/`high`/`max`, default `high`; aliases `minimal` → low, `ultra` → max; open weights 1–100). Don't pick a level in the prompt; don't write "answer without reasoning" (at most a `low` request).

**6. Length as a task requirement** — 1017 writes shorter by design, so a carried-over "be brief" may overcorrect (hypothesis to re-measure).

### Universal-DeepSeek checklist

- [ ] System prompt brief; placement decided by measurement when porting
- [ ] `<context>` with `relevance` attrs + `<instruction>`
- [ ] JSON demanded in prose (not `response_format` alone)
- [ ] `reasoning_content` handled by the client, not in prose
- [ ] Length stated per deliverable, no global brevity line
- [ ] No model name hardcoded

---

## Cross-vendor (three-vendor — Claude + GPT + Gemini)

The hardest classic case: an `AGENTS.md` or system prompt meant to work on **two or three frontier vendors** — Claude Code, Codex CLI, and/or Gemini CLI. The vendors have **opposite defaults** on several axes; cross-vendor wording must be acceptable to all targets, which usually means picking the stricter constraint.

### The three-vendor opposite-default table

| Axis | Claude default | GPT-5.x default | Gemini 3.x default | Cross-vendor wording |
|---|---|---|---|---|
| Process vs outcome | Tolerates step naming | Treats step prescription as noise (5.5 guide; GPT-6 guide outcome-first too) | Tolerates; CoT specifically hurts | Outcome-first wording; steps as "typical sub-tasks, choose order yourself" |
| Few-shot examples | 3-5 diverse examples = standard | Often hurts reasoning — zero-shot default | Mixed (Pro: less; Lite: more for format) | Skip few-shot unless format genuinely strict; if needed, 1-2 only |
| Tool guidance | System prompt is fine | Strongly prefer inside tool description | System instruction OK | Move tool guidance into tool descriptions (works for all) |
| Reasoning prompts | "Verify against criteria" works | `reasoning_effort` parameter | `thinking_level` parameter | Never "think step by step" / "think harder" in body — use API knob |
| **Persona / "act as"** | Functional 1-line role useful | **Outcome-first beats persona — strip** | **+5% reasoning boost — keep** | ⚠️ **CONTRADICTION** — see § "The persona problem" below |
| Subagent spawning | Opus 5 lineage / Fable delegate readily — boundaries + cap | Codex spawns what's defined | Spawns what's defined; supports remote subagents | Be explicit when you want delegation |
| Output format | Prose constraints work | Push to `json_schema` | Push to `response_json_schema` | Prefer schema for all three |
| Aggressive emphasis | Overuse → overtriggering on every current Claude | Mostly inert noise | Inert noise | Reserve ALL-CAPS / "CRITICAL:" for safety invariants only |
| **Sampling params in body** | **Every Claude: 400 at any value (API version `2026-10-21`)** | Tunable on 5.6; **GPT-6 Astra / 6.1 Sol: unsupported; 6.1 Terra / 6 Sol / 6 Luna: only at effort `none`** | **Ignored on Argon / 3.6+ (400 announced); defaults on earlier 3.x** | Don't reference sampling anywhere, and don't build variety on it — all three vendors have removed or pinned the lever. Use "propose N directions" |
| **Response length default** | **Opus 5 runs long and does not calibrate** (effort won't shorten it) — concision must be prompted | **5.6 is terser than 5.5** — a carried-over "be brief" overcorrects | 3.6 Flash ~17% terser than 3.5 Flash; 3.8 Flash works harder | ⚠️ **Live contradiction.** State length as a task requirement ("at most 5 bullets", "one paragraph per finding"); never as a global disposition |
| Instruction repetition | Tolerated; a short concision reminder near the end is recommended on Opus 5 | **Measured cost on 5.6** (leaner prompts: +10–15% score, −41–66% tokens) | Concise input is a family rule; verbose prompts get over-analyzed | Say each thing once, in its own section; if an echo is needed for Opus 5, keep it to one short line |
| **Negative constraint position** | Anywhere | Anywhere | **At end (drops early negatives)** | Place at end of file (strictest wins) |
| **XML + Markdown mixing** | Tolerated | Tolerated | **Pick one** | Pick one consistently (strictest wins) |
| **Blanket "do not" instructions** | Tolerated | Tolerated | **Over-indexes — fails basic logic** | Replace with positive scoped instructions ("Use the provided context for deductions") |
| CoT scaffolding | Tolerated | Hurts | **Hurts (and explicit anti-pattern)** | Strip — works for all three |

### The persona problem (the unique three-vendor contradiction)

**There is no neutral position on persona.** A "You are a senior engineer" line:
- **Helps Gemini 3** (+5% reasoning boost — Google research; not restated for Argon)
- **Hurts GPT** (5.5 guide + community consensus; the GPT-6 guide is silent — `gpt.md` rule #12)
- **Neutral on Claude** (works either way)

The `/prompt-atlas` skill **must detect this contradiction** and **ask the user which target(s) they actually have** before recommending strip-or-keep. See SKILL.md Step 4 (contradiction detection).

When the user can't or won't pick one target, three compromises exist:

**A. Methodological anchors only (recommended default).**

Replace credential-naming personas with **frame-naming** ones — these work on all three vendors because they specify *what to do*, not *who to be*:

| ❌ Strip these (credential-naming) | ✅ Keep these (frame-naming) |
|---|---|
| "You are a senior backend engineer with 15 years of experience" | "Apply systems-thinking lenses: 2nd/3rd-order effects, 10-year horizon" |
| "You are an expert at distributed systems" | "First identify the hardest sub-problem, then solve it before easier ones" |
| "Act as a Stanford-trained statistician" | "Reason from first principles; verify against the success criteria before finishing" |

Frame-naming gives Gemini its identity boost, gives Claude a useful constraint, and avoids GPT's persona penalty.

**B. Drop persona entirely (cost: -5% on Gemini).**

Acceptable when GPT is the primary target and Gemini is secondary.

**C. Conditional split (heaviest maintenance).**

`@import` a Gemini-only persona block. Only worth it when the +5% Gemini boost is measurably load-bearing.

### Other rules for cross-vendor `AGENTS.md`

**1. Outcome-first wording, not step prescription.**

> Fix the failing CI pipeline. Typical sub-tasks: read the most recent failed log, identify the failing job, propose a fix, run tests locally before pushing. Adapt as needed.

Works on all three vendors — outcome leads, steps are guidance.

**2. Watch the Codex 32 KiB cap.**

If `AGENTS.md` is shared with Codex, it's subject to `project_doc_max_bytes` (32 KiB default). Claude Code has no hard cap but suffers context rot past ~300 lines. Gemini CLI has no documented hard cap. **Strictest wins: target under 8 KiB load-bearing.**

**3. Use markdown headers as primary skeleton; XML only for inline content.**

Markdown is readable in version control and works on all three. Reserve XML for inline tagging.

**Gemini gotcha:** don't mix XML and Markdown in the same file — Gemini is strict, Claude/GPT tolerate it. Cross-vendor: pick one per file.

**4. Tool descriptions over system prompt for tool guidance.**

OpenAI's strong recommendation; Anthropic and Google both tolerate either. Pick OpenAI's stricter rule for cross-vendor.

**5. Avoid model-name pinning.**

"When using Opus 5.5, do X" / "GPT-6.1 Sol should Y" / "On Gemini 4 Argon, Z" defeats cross-vendor purpose. Functional descriptions only.

**6. Don't reference sampling parameters in body — and don't design around them.**

Every Claude rejects `temperature` / `top_p` / `top_k` at any value since API version `2026-10-21`; Gemini 4 and 3.6+ ignore them (400 announced; earlier 3.x keep defaults); GPT-6 Astra / 6.1 Sol don't accept them and the other GPT-6.x tiers only at effort `none`. Only GPT-5.6 still tunes freely. Don't write "use temperature 0.3" in any cross-vendor body, and don't build a prompt whose variety depends on the knob — steer it through wording ("propose N directions, then implement the chosen one").

**6a. Don't set a global length disposition.**

The vendors' defaults now point in **opposite** directions: Opus 5 runs long and won't calibrate, while GPT-5.6, Gemini 3.6 Flash and DeepSeek V4.1-Flash-1017 are terser than the versions they replace. "Be concise" starves the newer GPT/Gemini models; "be thorough" bloats them and does nothing useful for Opus 5. Express length where it's actually needed, as a requirement of the deliverable.

**7. Place negative constraints at the end of the file.**

Gemini drops early negatives; Claude and GPT tolerate them. Place at end (strictest wins).

**8. Replace blanket "do not" instructions with positive scoped instructions.**

Gemini over-indexes on "do not infer" / "do not guess" and may fail basic logic. Claude/GPT tolerate. Replace with: *"Use the provided context for deductions; if the answer isn't there, return 'I don't know'."* Works for all three.

### Cross-vendor checklist

- [ ] Outcome-first phrasing dominates; steps are guidance, not requirement
- [ ] No model-name pinning
- [ ] Persona resolved per persona-problem section above (anchors / drop / conditional)
- [ ] Tool-specific guidance migrated into tool descriptions
- [ ] No sampling parameters mentioned in body, and no behavior depending on them
- [ ] Length stated as a task requirement, not as a global disposition
- [ ] Anti-hallucination snippets explicit
- [ ] Total file size well under 32 KiB (target <8 KiB load-bearing)
- [ ] Aggressive emphasis used on safety invariants only
- [ ] No "think step by step" / "think harder" / CoT scaffolding in body
- [ ] Negative constraints at end of file
- [ ] No blanket "do not" — positive scoped only
- [ ] Markdown skeleton; XML only for inline content
- [ ] Output contracts pushed to schema where stack supports
- [ ] Universal-Claude (`claude.md`) + Universal-GPT (`gpt.md`) + Universal-Gemini (`gemini.md`) checklists all pass

When checklists conflict, the **three-vendor opposite-default table** above decides.

---

## Cross-vendor (4+)

A single prompt meant to run on any mix of four or more current frontier vendors — Claude, GPT, Gemini, Kimi, Z.ai GLM, frontier Qwen, DeepSeek, xAI Grok, Mistral, Meta Muse, Tencent Hunyuan, MiniMax / MiMo / StepFun (`lineup.md`). This is the hardest case in the skill and **usually a design mistake to attempt as a single universal artifact.**

### Why "universal across everything" is usually a trap

Each additional vendor adds at least one **opposite-default axis** that compromises wording across all the rest. The marginal cost of a +1th vendor isn't small — it's the cumulative product of all the strictness constraints. By the time you've satisfied:

- Codex's 32 KiB hard cap
- Gemini's "no temperature in body" + "negative constraints at end" + "no XML+Markdown mix"
- GPT's "strip persona / strip few-shot / outcome-first"
- GLM's "<4 KiB load-bearing + no identity pinning + reasoning-re-injection"
- DeepSeek's "user-prompt-priority + brief system"
- Qwen's "granular constraints + self-verify + numbered requirements"
- Kimi's "persona helps / few-shot helps" (which conflicts with GPT)

…you've written a prompt that doesn't optimally serve any of them, only minimally fails to break on all of them.

The recommended pattern instead:

1. **Pick a primary vendor** based on actual workload. Tune the artifact for it.
2. **For secondary vendors**, ship vendor-specific overrides (`AGENTS.openai.md`, `AGENTS.deepseek.md`, etc.) — most agentic systems have a convention for these.
3. **Reserve the base `AGENTS.md` for vendor-neutral facts** — repo conventions, tool descriptions, terminology, success criteria. Move the opinionated rules to overrides.

### When 4+ universal is genuinely required

A few real cases:
- **Cross-tool router** (e.g., user has Claude Code Router routing between Claude and GLM and DeepSeek based on task) — but routers usually inject vendor-specific shims, so the universal layer can stay vendor-neutral
- **Compliance / governance documents** that must be applied to every agent regardless of vendor — but these are usually principle-level (no specific wording)
- **Cross-team artifacts** where the team can't predict which model the consumer will use — but in 2026, most teams now know

If you genuinely have a 4+ case, the working pattern is:

### The 4+ universal compromise matrix

Strictest-constraint-wins across all current frontier vendors. **Cite the strictest vendor in comments next to each rule** so future maintainers can debug:

| Axis | Strictest constraint | Source vendor |
|---|---|---|
| Total load-bearing size | **<4 KiB** | GLM (heavy host suppression) |
| Persona | **functional only, no identity pinning, no credential naming** | GLM (identity confusion) + GPT (outcome-first) |
| Process steps | **outcome-first with steps as "typical sub-tasks, choose order yourself"** | GPT (step prescription is noise) + Gemini (CoT hurts) |
| Few-shot | **0-2 examples max** | GPT (5.5 guide: hurts reasoning) |
| Self-verification | **explicit "review for coverage" instruction present — unless the Opus 5 lineage or Tencent Hy4 (vendor-named over-verification) is in scope; then state verification as part of the task** | Qwen (load-bearing) vs Opus 5 / Hy4 (strip) |
| Reasoning depth | **API knob only, no prose CoT** | All frontier vendors (universal) |
| Output format | **`json_schema` / `json_object` API + JSON demanded in prose** | DeepSeek (both required) |
| Sampling params | **no mention in body, no behavior depending on them** | every Claude (400 at any value, API `2026-10-21`) + Gemini (ignored on Argon / 3.6+) + GPT-6 Astra / 6.1 Sol (unsupported; other 6.x only at `none`) + Kimi (fixed on hosted API) |
| Response length | **stated per deliverable ("at most 5 bullets"), never as a global disposition** | GPT-5.6 / Gemini 3.6 Flash / DeepSeek V4.1-Flash-1017 (already terse) vs Opus 5 (runs long, won't calibrate) — the two directions cancel |
| Instruction repetition | **each rule stated once** | GPT-5.6 (measured token/score cost) |
| Reasoning off-switch | **never write "answer without reasoning"** | Kimi K2.7-Code / K3 / K3.1 Pro, GLM-5.3 (all variants), Grok 4.5+, Claude Opus 5.5, Mistral Large 4, hosted Muse Wonder / Spark and Muse Spark Open, MiniMax M3.1-Flash, StepFun Step 5, Qwen3.8-Max open weights (`Qwen3.8-2.4T-A95B`) — thinking forced on / cannot be disabled; Opus 5 and Sonnet 5.5 `between_tools` (increases tag leakage) |
| Aggressive emphasis | **safety invariants only** | every current Claude (overtriggers) |
| Negative constraints | **at end of file, positive-scoped phrasing preferred** | Gemini (drops early; over-indexes on blanket negatives) |
| XML + Markdown | **markdown skeleton, XML inline only** | Gemini (strict) |
| Tool guidance | **inside tool description, not system body** | GPT + DeepSeek (system overuse penalty) |
| Instruction placement | **brief role in system + bulk in user message** | DeepSeek (user-prompt priority) — costs little on others |
| Image content placement | **image blocks in user messages only** — the safe, standard placement on every multimodal vendor | convention (the DeepSeek vision-model 400 that motivated it is archived) |
| Unverified new-family axes | **don't claim behavioral deltas for vendors without prompting guides — apply the strictest verified neighbor and mark `?`** | Grok 4.x, Gemini 4 Argon, Mistral Medium 3.5, hosted Muse Wonder / Spark, Tencent Hy4 preview, MiniMax M3.1-Flash, MiMo-V2.6, Step 5 Preview (no guides published) — anti-fabrication |
| Granularity | **explicit constraints, hex colors, sizes, scopes** | Qwen (vagueness hurts) — costs little on others |
| Numbered requirements | **A, B, C, D rather than "the following considerations"** | Qwen (skips un-emphasized) — costs nothing on others |
| Identity check | **don't ask the model to name itself** | GLM (distillation artifact) |
| Reasoning re-injection | **`reasoning_effort` where the router forwards it; explicit "write reasoning before answering" markers only as fallback** | GLM (host prompt suppression) — costs little on others |
| Reasoning content round-trip | **client library handles** | DeepSeek (mandatory with tools), Kimi K3 line, MiMo, Mistral Large 4 (full-message replay) |

Following all of these produces a prompt that's safe on every current frontier vendor but isn't optimal on any of them. That's the trade-off.

### Cross-vendor (4+) checklist

- [ ] All single-vendor universal checklists pass (Claude, GPT, Gemini, Kimi, GLM, frontier Qwen, DeepSeek as applicable)
- [ ] Total load-bearing size <4 KiB
- [ ] Persona is functional-role only, no identity pinning, no credential naming
- [ ] No CoT scaffolding; reasoning lever is API knob
- [ ] Output contract is `json_schema`/`json_object` AND JSON demanded in prose
- [ ] No sampling parameters in body; variety comes from "propose N directions"
- [ ] Length stated per deliverable; no global "be brief" / "be thorough"
- [ ] Each rule stated once (no cross-section restatement)
- [ ] No "answer without reasoning" anywhere in the body
- [ ] Negative constraints at end, positive-scoped
- [ ] Markdown skeleton, XML inline only
- [ ] Tool guidance in tool descriptions
- [ ] Brief system prompt + bulk in user message
- [ ] Granular constraints (hex colors, sizes, scopes — not vibes)
- [ ] Numbered requirements
- [ ] Self-verification instruction present (task-phrased if Opus 5.x / Hy4 is in scope)
- [ ] No identity-asking
- [ ] GLM reasoning lever handled under heavy host prompts (param first, re-injection fallback)
- [ ] `reasoning_content` round-trip handled by client

When the user pushes back on the 4+ constraint set ("but this prompt is too long now / doesn't sound smart"), the answer is: **you don't have a 4+ universal case; you have a primary-vendor case with overrides for the rest.** Recommend that pattern.

---

## Cross-model wording principles (apply universally)

These hold regardless of which model or vendor you're targeting:

### Use the API knob over wording for reasoning depth

`effort` (Claude) / `reasoning_effort` (GPT and most others) / `thinking_level` (Gemini) is the load-bearing lever (`lineup.md`). The one documented prose control is Muse Spark Open's `Reasoning strength: N.` line (`meta.md`), valid for that model only. Wording-based reasoning prompts ("think step by step", "be thorough", "think harder") work inconsistently across models. The API knob is consistent.

### Prefer "verify" / "evaluate" / "consider" over "think"

"Verify your answer against these criteria before finishing" works. "Think harder" doesn't.

### Aggressive emphasis is for invariants only

Current Claude overtriggers; GPT is inert noise. Either way, save it for things where wrong = catastrophic (secrets, data loss, destructive operations).

### Prompts age fast on the GPT side

OpenAI ships frequent minor versions and prompting style shifts more between them than Claude generations do. A GPT prompt that's 6 months old should get a fresh-eyes review on every model upgrade. Anthropic's versions are forward-compatible enough not to need this.

### Document the WHY

Across all vendors, rules with reasons generalize better than rules without. "We do X because Y" beats "do X". The model uses Y to handle edge cases.
