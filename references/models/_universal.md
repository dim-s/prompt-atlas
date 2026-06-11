# Universal / multi-model / cross-vendor wording rules

Rules for prompts that must work across multiple models or vendors. The differences themselves live in `matrix.md`. The shared principles live in `principles.md`. This file covers the **compromises** — what to write when the prompt has to be acceptable to several models with conflicting defaults.

---

## When this file applies

Multiple overlapping cases. Read the relevant section.

| Case | Section |
|---|---|
| Prompt must work across multiple Claude versions (Fable 5 + Opus 4.8 / 4.7 + Sonnet 4.6 + Haiku 4.5) | § Universal Claude |
| Prompt must work across multiple GPT-5.x versions (5.3 + 5.4 + 5.5) | § Universal GPT-5.x |
| Prompt must work across multiple Gemini 3.x variants (Pro + Flash + Flash-Lite) | § Universal Gemini |
| Prompt must work across two or three frontier vendors (Claude + GPT-5.x + Gemini) | § Cross-vendor (three-vendor) |
| Prompt must work across multiple Kimi K2.x versions | § Universal Kimi |
| Prompt must work across multiple Z.ai GLM versions | § Universal GLM |
| Prompt must work across multiple frontier Qwen versions | § Universal frontier Qwen |
| Prompt must work across multiple DeepSeek V3/V4 versions | § Universal DeepSeek |
| Prompt must work across four-or-more frontier vendors (any mix of Claude / GPT / Gemini / Kimi / GLM / Qwen / DeepSeek) | § Cross-vendor (4+) — and read the warning |

Most production CLAUDE.md / AGENTS.md fall into one of these — pure single-model prompts are rarer than they look. **4+ vendor cross-everything universal prompts are usually a trap** — see § Cross-vendor (4+) before committing to the design.

---

## Universal Claude

### How to ask the user about target

If unclear, one short question:

> *"Под какую модель этот промпт — Fable 5, Opus 4.8 / 4.7, Sonnet 4.6, Haiku 4.5 или универсальный (должен работать на всех сразу)?"*

Default to **universal** when in doubt — write for lowest-common-denominator behavior across 4.5+ models.

### Rules

**1. Write for the least forgiving model in the set.**

If Haiku 4.5 is in scope, be MORE specific — Haiku handles vague prompts worse than Opus. Spell things out concretely; add examples where format matters; name tools explicitly when you expect them to be used.

**2. Avoid leaning on single-model features.**

- Don't assume Haiku will do deep multi-step reasoning — break complex tasks into concrete steps.
- Don't assume Opus's literalism saves you from stating scope (Sonnet generalizes more freely).
- Don't write tone instructions assuming one model's default — "be less formal" reads differently on Opus 4.7 vs Haiku.
- Don't write subagent guidance assuming one direction — the default **diverges inside the family**: Fable 5 delegates readily, Opus 4.8/4.7 undertrigger. Universal wording states *when delegation is appropriate* ("delegate independent subtasks; work directly for single-file reads") — that reads as a boundary on Fable 5 and as encouragement on Opus.

**3. Be moderate with emphasis.**

`CRITICAL:` and `YOU MUST` are overread on 4.5+ and underread on older models. Avoid them as much as possible. When you need emphasis, use it sparingly and only on genuine invariants.

**4. Use structure rather than intensity.**

Instead of fighting for attention with ALL-CAPS, use XML tags or clear section headings. Structure is model-invariant; emotional intensity is not.

```xml
<invariants>
- Never commit .env or secrets.* files
- Always run the linter before committing
</invariants>

<style>
- Use ES modules, not CommonJS
- snake_case for Python
</style>
```

**5. Prefer positive framing and WHY-based rules.**

Negative-only rules without alternatives and unexplained mandates are fragile.

**6. Watch for model-specific scaffolding that should be stripped.**

- Old "avoid AI slop" long frontend prompts → trim for 4.7
- "After every 3 tool calls, summarize" → 4.7 already does this; harmless on 4.6 but adds noise
- "Be thorough, use X tool when in doubt" → causes overtriggering on 4.5+; soften to "Use X when it enhances your understanding"

**7. Don't name a specific model in the prompt.**

Use functional roles: "You are a helpful coding assistant" — not "You are Claude Opus 4.7".

**8. When a universal prompt isn't enough, split.**

Baseline for the weakest model + an "additional guidance" section that cites stronger-model behaviors without mandating them:

> Additional: if you have adaptive thinking or extended reasoning available, use it to verify your answer against the success criteria before responding. If not, quickly re-read the question and your answer one more time before finalizing.

### Universal-Claude checklist

- [ ] No hard dependency on model-specific features (adaptive thinking, high-res vision, `xhigh` effort)
- [ ] Specificity high enough for Haiku 4.5 to follow
- [ ] Explicit scope stated for Opus 4.7's literalism
- [ ] Overengineering / over-exploration guards for Sonnet 4.6
- [ ] Emphasis used sparingly (no "CRITICAL:" stack)
- [ ] No model name hardcoded
- [ ] Structure (XML/headings) used to signal important parts, not emphasis
- [ ] If thinking-related: phrased as "consider" / "verify" / "reason through" rather than "think harder"
- [ ] No "echo / show / explain your reasoning" instructions — refusal trigger (`reasoning_extraction`) when Fable 5 is in scope; unnecessary on the rest

---

## Universal GPT-5.x

A prompt that must work across two or more 5.x versions (e.g., a Codex agent prompt that runs on 5.3-codex *and* 5.5).

### Rules

**1. Write for the most literal version in scope.**

If 5.5 is in scope, default to outcome-first phrasing — earlier versions tolerate it; 5.5 punishes the alternative.

**2. Don't lean on 5.5-only features.**

- Don't assume 1M-token-class context behavior — 5.3 and earlier are tighter
- Don't assume the lowest hallucination rate — keep anti-hallucination snippets
- Don't strip few-shot examples that are load-bearing on 5.3 / 5.4 just because 5.5 prefers zero-shot. Keep them and trust 5.5 to ignore.

**3. Push output format into `json_schema` if the stack supports it.**

Most version-neutral lever. The system prompt becomes leaner regardless of which version reads it.

**4. Avoid hardcoding model names.**

"You are GPT-5.5" locks you to the model. Functional roles: "You are a code-review agent."

**5. Use `reasoning_effort` rather than wording for reasoning depth.**

Wording-based reasoning prompts ("think step by step") work inconsistently across versions. The API knob is consistent.

### Universal-GPT-5.x checklist

- [ ] No hard dependency on 5.5-only context length (>200K) or 5.5-only hallucination rate
- [ ] Output contract is in `json_schema` if stack supports it; otherwise format described once, not via few-shot
- [ ] Reasoning depth configured via `reasoning_effort`, not "think step by step" in prose
- [ ] No 5.4-era process-step prescription that 5.5 would treat as noise (or it's tagged "step suggestions, not requirements")
- [ ] No model name hardcoded
- [ ] Tool-specific guidance inside tool descriptions, not system prompt
- [ ] Stable content first; volatile / user-specific content at the end

---

## Universal Gemini

A prompt that works across Gemini 3.1 Pro + 3 Flash + 3.1 Flash-Lite.

### Rules

**1. Concise wording is critical** — Flash-Lite needs it most for latency.

**2. No CoT scaffolding** — all three Gemini 3.x variants internalize reasoning via `thinking_level`.

**3. Identity-based persona helps all three** — add a 1-line "You are a [role]" — measurable boost.

**4. Don't mention temperature in the body** — all three default to 1.0 and Google explicitly recommends not lowering.

**5. Pick XML or Markdown, not both** — stricter than Claude/GPT.

**6. Negative constraints at the end** — model drops early negatives.

**7. Output format via `response_json_schema`** when stack supports.

**8. Tool guidance inside tool descriptions**, not system prompt.

**9. No model name hardcoded.**

### Universal-Gemini checklist

- [ ] Concise (Flash-Lite-friendly)
- [ ] No CoT scaffolding
- [ ] No temperature in body
- [ ] Identity-based persona present
- [ ] XML or Markdown — not both
- [ ] Negative constraints at end
- [ ] `json_schema` for structured output
- [ ] Tool guidance in tool descriptions
- [ ] No model name hardcoded
- [ ] If long context: data top, question end, "Based on the above..." anchor

---

## Universal Kimi

A prompt that works across Kimi K2.6 and (still-deployed) K2.5. K2 itself retires May 25, 2026 — if you see K2 in scope, the prompt is due for migration, not universalization.

### Rules

**1. Write for K2.6's multimodal default.** Don't write "this is a text-only model" assumptions — K2.6 supports vision (MoonViT). K2.5 was text-only; assumption that worked on K2.5 will be wrong on K2.6.

**2. Agent Swarms is K2.6-only at scale.** K2.5 supports 100 sub-agents / 1500 steps; K2.6 raises to 300 / 4000. Don't write swarm prompts that depend on K2.6's higher bounds when K2.5 is in scope.

**3. Two-mode design (Thinking/Instant) consistent across the family.** Don't hardcode temperature; let `extra_body.thinking` toggle do the work.

**4. Persona helps both versions** — keep functional persona lines.

**5. Few-shot helps both versions** — 3–5 examples for format steering is family-safe.

### Universal-Kimi checklist

- [ ] No K2.6-only swarm scale assumed (300 sub-agents, 4000 steps)
- [ ] No "text-only" assumption (K2.6 is multimodal)
- [ ] Functional persona present (helps both K2.5 and K2.6)
- [ ] No temperature hardcoded in body
- [ ] Tool calling described as standard OpenAI tools format
- [ ] No "think step by step" in body — use `extra_body.thinking` parameter

---

## Universal GLM

A prompt that works across GLM-5.1, GLM-5, and (still-deployed) GLM-4.6. The family-wide rule about heavy host system prompts suppressing thinking applies to all three — but most strongly on the larger models, which have more capacity to be "interfered with."

### Rules

**1. Keep host-side overhead in mind.** If the prompt runs through Claude Code Router, OpenCode, Cline, or similar — every kilobyte stacks on top of the router's injection. Target <4 KiB load-bearing for GLM-routed deployments.

**2. Don't pin model identity.** All three versions have the "I am Claude" distillation artifact. Use functional roles only.

**3. Inject explicit reasoning directives** when the prompt runs under a heavy host. Custom `<reasoning_content>` markers + "write detailed reasoning before answering" works across the family.

**4. `json_schema` preferred** over `json_object` — works on all three; gives stricter shape control.

**5. Outcome-first beats step prescription** — process steps tolerated but unnecessary.

### Universal-GLM checklist

- [ ] Total system + AGENTS.md size <4 KiB load-bearing
- [ ] No identity pinning ("You are GLM-5.1") — functional roles only
- [ ] Reasoning re-injection present if running under heavy host system prompt
- [ ] `json_schema` over `json_object` for structured output
- [ ] No "think step by step" in body
- [ ] Long-horizon framing where applicable (8-hour autonomous on 5.1)
- [ ] No model name hardcoded

---

## Universal frontier Qwen

A prompt that works across Qwen3.7-Max, 3.7 Plus, 3.6 Plus, 3.6 Max-Preview, and (still-deployed) Qwen3-Max-Thinking. **Frontier Qwen only** — small-local Qwen 2-9B has its own universal section in `small-local.md`.

### Rules

**1. Write for the most granular target in scope.** Qwen 3.7 Plus is more likely than 3.7-Max to skip un-emphasized sections — write for Plus, accept overhead on Max.

**2. Add self-verification.** Free on other vendors, load-bearing on Qwen. Always include "before finishing, review your response and check that every requested element is covered."

**3. Number requirements explicitly.** Avoid "address the following considerations" when you mean "address each of: A, B, C, D."

**4. Don't assume 1M context reliability.** Validate before committing to long-context patterns. The Max-Preview's 256K context is the safest assumption for cross-version Qwen prompts.

**5. Thinking mode toggle is the lever.** Don't write "think step by step"; set the toggle.

**6. Granular constraints over inferred intent** — restore detail stripped during GPT-5.5 tuning.

### Universal-frontier-Qwen checklist

- [ ] Constraints explicit (hex colors, sizes, ratios, scopes)
- [ ] Self-verification instruction present
- [ ] Requirements numbered, not bulleted prose
- [ ] No assumption of 1M context reliability
- [ ] Thinking mode toggle assumed, not prose
- [ ] No CoT scaffolding in body
- [ ] No model name hardcoded
- [ ] If multimodal-curious: confirm vision support before assuming (3.7-Max is text-only)

---

## Universal DeepSeek

A prompt that works across DeepSeek V4-Pro, V4-Flash, and (still-deployed) V3.2. The V3 → V4 break is **API-level** (`reasoning_content` round-trip semantics flip) — wording is mostly forward-compatible.

### Rules

**1. User-prompt priority is family-wide.** Always — V3 and V4 both prefer instructions in user prompt over system prompt.

**2. Brief system prompt.** "You are a senior architect" style. Long system prompts are the #1 failure mode across the family.

**3. XML-tagged context with `relevance` attrs.** ~92% vs ~45% task success; both V3 and V4 reward the structure.

**4. Demand JSON in prose AND set `response_format`.** Both, not either. Family-wide.

**5. Don't reference `reasoning_content`** in prose — the API behavior differs between V3 and V4; client-library should handle.

**6. Tool calls in thinking mode work on V4 only.** If V3 is in scope, don't write prompts that depend on mixing reasoning and tool calls in the same response.

**7. Three-level thinking lever is V4-only** (`off`/`high`/`max`). V3.2 / R1 used a single-toggle. Don't write prompts that pick a specific level — set via API config.

### Universal-DeepSeek checklist

- [ ] System prompt is brief (one-line role)
- [ ] Bulk of instruction lives in user prompt
- [ ] User-prompt structure uses `<context>` with `relevance` attrs and `<instruction>`
- [ ] JSON output demanded in prose (not relying on `response_format` alone)
- [ ] No reasoning + tool-call interleaving assumed (V3 doesn't support; V4 does)
- [ ] `reasoning_content` round-trip handled by client, not mentioned in prose
- [ ] No model name hardcoded

---

## Cross-vendor (three-vendor — Claude + GPT + Gemini)

The hardest classic case: an `AGENTS.md` or system prompt meant to work on **two or three frontier vendors** — Claude Code, Codex CLI, and/or Gemini CLI. The vendors have **opposite defaults** on several axes; cross-vendor wording must be acceptable to all targets, which usually means picking the stricter constraint.

### The three-vendor opposite-default table

| Axis | Claude default | GPT-5.x default | Gemini 3.x default | Cross-vendor wording |
|---|---|---|---|---|
| Process vs outcome | Tolerates / appreciates step naming, esp. Opus 4.7 | Treats step prescription as noise, esp. 5.5 | Tolerates; CoT specifically hurts | Outcome-first wording; steps as "typical sub-tasks, choose order yourself" |
| Few-shot examples | 3-5 diverse examples = standard | Often hurts reasoning — zero-shot default | Mixed (Pro: less; Lite: more for format) | Skip few-shot unless format genuinely strict; if needed, 1-2 only |
| Tool guidance | System prompt is fine | Strongly prefer inside tool description | System instruction OK | Move tool guidance into tool descriptions (works for all) |
| Reasoning prompts | "Verify against criteria" works | `reasoning_effort` parameter | `thinking_level` parameter | Never "think step by step" / "think harder" in body — use API knob |
| **Persona / "act as"** | Functional 1-line role useful | **Outcome-first beats persona — strip** | **+5% reasoning boost — keep** | ⚠️ **CONTRADICTION** — see § "The persona problem" below |
| Subagent spawning | Opus 4.7 spawns fewer | Codex spawns what's defined | Spawns what's defined; supports remote subagents | Be explicit when you want delegation |
| Output format | Prose constraints work | Push to `json_schema` | Push to `response_json_schema` | Prefer schema for all three |
| Aggressive emphasis | Overuse → overtriggering 4.5+ | Mostly inert noise | Inert noise | Reserve ALL-CAPS / "CRITICAL:" for safety invariants only |
| **Temperature in body** | Tunable; mentioning OK | Tunable; mentioning OK | **Don't tune (1.0 fixed); don't mention** | Don't reference temperature in body — let API config handle per vendor |
| **Negative constraint position** | Anywhere | Anywhere | **At end (drops early negatives)** | Place at end of file (strictest wins) |
| **XML + Markdown mixing** | Tolerated | Tolerated | **Pick one** | Pick one consistently (strictest wins) |
| **Blanket "do not" instructions** | Tolerated | Tolerated | **Over-indexes — fails basic logic** | Replace with positive scoped instructions ("Use the provided context for deductions") |
| CoT scaffolding | Tolerated | Hurts | **Hurts (and explicit anti-pattern)** | Strip — works for all three |

### The persona problem (the unique three-vendor contradiction)

**There is no neutral position on persona.** A "You are a senior engineer" line:
- **Helps Gemini 3** (+5% reasoning boost — Google research)
- **Hurts GPT-5.5** (community consensus + OpenAI guidance)
- **Neutral on Claude** (works either way)

The `/prompt-atlas` skill **must detect this contradiction** and **ask the user which target(s) they actually have** before recommending strip-or-keep. See SKILL.md § Contradiction detection.

When the user can't or won't pick one target, three compromises exist:

**A. Methodological anchors only (recommended default).**

Replace credential-naming personas with **frame-naming** ones — these work on all three vendors because they specify *what to do*, not *who to be*:

| ❌ Strip these (credential-naming) | ✅ Keep these (frame-naming) |
|---|---|
| "You are a senior backend engineer with 15 years of experience" | "Apply systems-thinking lenses: 2nd/3rd-order effects, 10-year horizon" |
| "You are an expert at distributed systems" | "First identify the hardest sub-problem, then solve it before easier ones" |
| "Act as a Stanford-trained statistician" | "Reason from first principles; verify against the success criteria before finishing" |

Frame-naming gives Gemini its identity boost, gives Claude a useful constraint, and avoids GPT-5.5's persona penalty.

**B. Drop persona entirely (cost: -5% on Gemini).**

Acceptable when GPT-5.5 is the primary target and Gemini is secondary.

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

"When using Opus 4.7, do X" / "GPT-5.5 should Y" / "On Gemini 3 Pro, Z" defeats cross-vendor purpose. Functional descriptions only.

**6. Don't reference temperature in body.**

Gemini requires 1.0; Claude and GPT-5.x are tunable. Don't write "use temperature 0.3" in any cross-vendor body — let each vendor's API config handle.

**7. Place negative constraints at the end of the file.**

Gemini drops early negatives; Claude and GPT tolerate them. Place at end (strictest wins).

**8. Replace blanket "do not" instructions with positive scoped instructions.**

Gemini over-indexes on "do not infer" / "do not guess" and may fail basic logic. Claude/GPT tolerate. Replace with: *"Use the provided context for deductions; if the answer isn't there, return 'I don't know'."* Works for all three.

### Cross-vendor checklist

- [ ] Outcome-first phrasing dominates; steps are guidance, not requirement
- [ ] No model-name pinning
- [ ] Persona resolved per persona-problem section above (anchors / drop / conditional)
- [ ] Tool-specific guidance migrated into tool descriptions
- [ ] No temperature mentioned in body (leave to API config)
- [ ] Anti-hallucination snippets explicit
- [ ] Total file size well under 32 KiB (target <8 KiB load-bearing)
- [ ] Aggressive emphasis used on safety invariants only
- [ ] No "think step by step" / "think harder" / CoT scaffolding in body
- [ ] Negative constraints at end of file
- [ ] No blanket "do not" — positive scoped only
- [ ] Markdown skeleton; XML only for inline content
- [ ] Output contracts pushed to schema where stack supports
- [ ] Universal-Claude + Universal-GPT-5.x + Universal-Gemini checklists all pass

When checklists conflict, the **three-vendor opposite-default table** above decides.

---

## Cross-vendor (4+)

A single prompt meant to run on any mix of four or more current frontier vendors — Claude, GPT-5.x, Gemini 3.x, Kimi K2.6, Z.ai GLM, frontier Qwen, DeepSeek V4. This is the hardest case in the skill and **usually a design mistake to attempt as a single universal artifact.**

### Why "universal across everything" is usually a trap

Each additional vendor adds at least one **opposite-default axis** that compromises wording across all the rest. The marginal cost of a +1th vendor isn't small — it's the cumulative product of all the strictness constraints. By the time you've satisfied:

- Codex's 32 KiB hard cap
- Gemini's "no temperature in body" + "negative constraints at end" + "no XML+Markdown mix"
- GPT-5.5's "strip persona / strip few-shot / outcome-first"
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
| Persona | **functional only, no identity pinning, no credential naming** | GLM (identity confusion) + GPT-5.5 (outcome-first) |
| Process steps | **outcome-first with steps as "typical sub-tasks, choose order yourself"** | GPT-5.5 (step prescription is noise) + Gemini (CoT hurts) |
| Few-shot | **0-2 examples max** | GPT-5.5 (hurts reasoning) |
| Self-verification | **explicit "review for coverage" instruction present** | Qwen (load-bearing) |
| Reasoning depth | **API knob only, no prose CoT** | All frontier vendors (universal) |
| Output format | **`json_schema` / `json_object` API + JSON demanded in prose** | DeepSeek (both required) |
| Temperature | **no mention in body** | Gemini (1.0 fixed) |
| Aggressive emphasis | **safety invariants only** | Claude 4.5+ (overtriggers) |
| Negative constraints | **at end of file, positive-scoped phrasing preferred** | Gemini (drops early; over-indexes on blanket negatives) |
| XML + Markdown | **markdown skeleton, XML inline only** | Gemini (strict) |
| Tool guidance | **inside tool description, not system body** | GPT-5.5 + DeepSeek (system overuse penalty) |
| Instruction placement | **brief role in system + bulk in user message** | DeepSeek (user-prompt priority) — costs little on others |
| Granularity | **explicit constraints, hex colors, sizes, scopes** | Qwen (vagueness hurts) — costs little on others |
| Numbered requirements | **A, B, C, D rather than "the following considerations"** | Qwen (skips un-emphasized) — costs nothing on others |
| Identity check | **don't ask the model to name itself** | GLM (distillation artifact) |
| Reasoning re-injection | **explicit "write reasoning before answering" markers** | GLM (host prompt suppression) — costs little on others |
| Reasoning content round-trip | **client library handles** | DeepSeek V4 (mandatory) |

Following all of these produces a prompt that's safe on every current frontier vendor but isn't optimal on any of them. That's the trade-off.

### Cross-vendor (4+) checklist

- [ ] All single-vendor universal checklists pass (Claude, GPT-5.x, Gemini, Kimi, GLM, frontier Qwen, DeepSeek as applicable)
- [ ] Total load-bearing size <4 KiB
- [ ] Persona is functional-role only, no identity pinning, no credential naming
- [ ] No CoT scaffolding; reasoning lever is API knob
- [ ] Output contract is `json_schema`/`json_object` AND JSON demanded in prose
- [ ] No temperature in body
- [ ] Negative constraints at end, positive-scoped
- [ ] Markdown skeleton, XML inline only
- [ ] Tool guidance in tool descriptions
- [ ] Brief system prompt + bulk in user message
- [ ] Granular constraints (hex colors, sizes, scopes — not vibes)
- [ ] Numbered requirements
- [ ] Self-verification instruction present
- [ ] No identity-asking
- [ ] Reasoning re-injection present if running under heavy host prompts
- [ ] `reasoning_content` round-trip handled by client

When the user pushes back on the 4+ constraint set ("but this prompt is too long now / doesn't sound smart"), the answer is: **you don't have a 4+ universal case; you have a primary-vendor case with overrides for the rest.** Recommend that pattern.

---

## Cross-model wording principles (apply universally)

These hold regardless of which model or vendor you're targeting:

### Use the API knob over wording for reasoning depth

`effort` (Claude) / `reasoning_effort` (GPT-5.x) is the load-bearing lever. Wording-based reasoning prompts ("think step by step", "be thorough", "think harder") work inconsistently across models. The API knob is consistent.

### Prefer "verify" / "evaluate" / "consider" over "think"

"Verify your answer against these criteria before finishing" works. "Think harder" doesn't.

### Aggressive emphasis is for invariants only

Claude 4.5+ overtriggers; GPT-5.x is inert noise. Either way, save it for things where wrong = catastrophic (secrets, data loss, destructive operations).

### Prompts age fast on the GPT side

OpenAI ships frequent minor versions and prompting style shifts more between them than Claude generations do. A GPT prompt that's 6 months old should get a fresh-eyes review on every model upgrade. Anthropic's versions are forward-compatible enough not to need this.

### Document the WHY

Across all vendors, rules with reasons generalize better than rules without. "We do X because Y" beats "do X". The model uses Y to handle edge cases.
