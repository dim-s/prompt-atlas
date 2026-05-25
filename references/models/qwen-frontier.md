# Model-specific wording differences — Alibaba Qwen frontier family

What changes about how you should PHRASE prompts for **frontier-class Qwen models**: Qwen3.7-Max (May 2026), Qwen3.7 Plus, Qwen3.6 Plus / Max-Preview, and the Qwen3-Max-Thinking lineage. Companion to `claude.md`, `gpt.md`, `gemini.md`, `kimi.md`, `glm.md`, `deepseek.md`.

**This file covers frontier Qwen only.** Small-local Qwen variants (Qwen3 2B / 4B / e2b / e4b that run on LM Studio, Ollama, llama.cpp, vLLM at consumer hardware tiers) are covered in `small-local.md § Qwen` — different prompting regime, different failure modes, different reference matrix (`matrix-small.md`). When a user names just "Qwen" without size or tier, ask which one before applying advice from either file.

Frontier Qwen runs via **DashScope** (Alibaba Cloud's model API, `dashscope-intl.aliyuncs.com`), the Qwen Chat web UI, or through cross-tool routers via the OpenAI-compat surface. As of Qwen3.7-Max, Alibaba's flagship is **closed-weights** — a deliberate policy shift from the more open Qwen3 / Qwen3.6 family.

---

## Family-wide rules (apply to all current frontier Qwen versions)

These hold across Qwen3.6 Plus → Qwen3.6-Max-Preview → Qwen3-Max-Thinking → Qwen3.7-Max.

### 1. Granularity beats inference — more explicit than Claude/GPT/Gemini

Hands-on testing reports (and Qwen-team-style documentation) consistently flag that **vague instructions underperform** more sharply on Qwen than on Claude or GPT.

Where Claude infers from "make the hero section look modern," Qwen wants explicit hex colors, font sizes, padding values, accessibility ratios. Where Claude tolerates "improve the function," Qwen wants explicit success criteria.

This is **opposite to GPT-5.5's "outcome-first / let the model pick"** approach. Qwen wants both — outcome **and** explicit constraints. When porting a GPT-5.5 prompt to Qwen, restore the constraint detail you stripped.

**Wording fix when you see vagueness in a Qwen-targeted prompt:**

| Weak | Strong |
|---|---|
| "Modernize the design." | "Modernize the design: use color palette `#1a1a2e / #16213e / #0f3460 / #e94560` for primary surfaces; minimum 4.5:1 contrast; 16/24/32px spacing scale; max two font weights." |
| "Refactor for clarity." | "Refactor `foo.ts:foo()`: extract early-returns; replace nested ternary with switch; keep public API identical (verified by snapshot test)." |

### 2. Skips un-emphasized sections — anti-pattern: implicit-critical

Documented quirk: Qwen3.7 occasionally **omits entire sections** of a multi-section task if those sections weren't given explicit emphasis. The behavior is more pronounced on Qwen3.7 Plus than Max, but appears on both.

Wording mitigation:
- **Number every required section** in the request ("Section 1, Section 2, Section 3 — all must be present")
- **Add a self-verification step** ("Before responding, list each requested section and confirm coverage")
- **Use explicit emphasis sparingly but deliberately** for must-not-miss requirements

This is the opposite-direction of Claude 4.5+ where aggressive emphasis overtriggers; on Qwen it's the bare minimum to ensure attention.

### 3. Thinking mode is required-on for best results, with latency cost

Both Max and Plus variants of Qwen3.7 are documented as "**requiring** thinking mode enabled" for the quality numbers Alibaba publishes. The toggle is exposed in the Qwen Chat UI and as an API parameter on DashScope.

Cost: thinking mode adds **45–60 seconds latency** for complex outputs and generates substantially more tokens (~97 M across benchmarks vs ~24 M for comparable closed models).

**Wording implication:**
- Don't write "think step by step" — set the toggle, not the prose
- When latency matters (interactive UI, autocomplete), disable thinking and accept lower quality
- For agentic workflows: enable thinking, budget for the latency, expect quality

### 4. Iterative refinement beats one-shot rewrites

Hands-on testing: Qwen responds **better to targeted follow-up prompts** than to "rewrite this from scratch" requests. "Add the missing Phase 2 section" or "Increase hero text contrast" beats "redo the whole page."

Wording-wise this isn't a system-prompt rule — it's a session-shape recommendation. For an agentic prompt that runs autonomous loops, structure success criteria around verifiable sub-goals rather than holistic quality scores.

### 5. Self-verification prompts boost completeness

Per hands-on reports, adding *"Before finishing, review your response and check that every requested element is covered"* meaningfully improves output completeness. This is a free, low-cost addition for Qwen-targeted prompts.

Other vendors tolerate self-verification without much effect; on Qwen it's load-bearing.

### 6. Higher abstention rate — "I don't know" more often

Qwen3.7-Max has a **lower hallucination rate** than competitors (Alibaba reports −21.3 pp on AA-Omniscience hallucination), but at the cost of accuracy (−7.6 pp): the model is **choosing to say "I don't know"** more often rather than recalling more facts.

Abstention rate fell from 67.3% to 48.0% — lowest among frontier models tested. The model is doing **less guessing**.

**Wording implication:**
- Don't rely on broad factual recall ("name the author of X") — Qwen may abstain even on facts other models would confidently answer
- For knowledge-base tasks, supply reference text and instruct citation (matches Claude/Kimi long-context patterns)
- For reasoning tasks, the lower hallucination is a feature — leverage it by not pre-asserting facts in the prompt

### 7. Visual / aesthetic polish lags Claude / GPT

UI/UX outputs from Qwen are "functional but not always aesthetically refined." Animations are jittery; transitions incomplete; polish trails Claude Opus 4.7 and GPT-5.5 noticeably.

If the prompt is for visual / design work and aesthetic quality is the goal, Qwen may not be the right target. Flag this in reviews when the prompt is design-heavy and target is Qwen.

### 8. Native OpenAI + Anthropic API compatibility

Qwen3.7-Max exposes both an OpenAI-compatible surface and an Anthropic-compatible surface on DashScope. The wording inside the message body doesn't change between surfaces — the same prompt works on either client.

The choice matters at the SDK level, not the wording level. For prompt-atlas purposes, treat Qwen prompts as portable across the two API surfaces.

### 9. Long-context: 1M nominal, no independent verification at scale

Qwen3.7-Max advertises a **1 M token context window** (doubled from Qwen3.6-Max-Preview's 256 K). Alibaba's internal long-context evaluations are not independently verified yet (per public analysis as of May 2026).

**Wording rule:** treat 1 M context as **available but unverified**. For workloads that genuinely need >500 K, validate against your task before committing the prompt design to long-context patterns. Don't write prompts that assume reliable retrieval across the full window without measuring.

### 10. Closed-weights flagship, open-weights derivatives — pricing context

Qwen3.7-Max is **closed-weights**. Smaller Qwen3.6 derivatives (Qwen3.6 Plus, certain training-checkpoint releases) remain open on Hugging Face. This affects review economics — Qwen3.7-Max self-host isn't an option; you're paying DashScope rates. Prompts that target Qwen3.7-Max should not assume the cost-flexibility of GLM-5.1 / Kimi K2.6 / DeepSeek V4.

---

## Qwen3.7-Max (Alibaba, May 20-21, 2026 — current frontier)

### Headline facts

- **Closed-weights**, proprietary reasoning model optimized for autonomous agent execution
- **Context:** 1 M tokens (text-only — no image input on Max)
- **Performance:** Artificial Analysis Intelligence Index 56.6 (5th overall — behind GPT-5.5 at 60.2, Opus 4.7 at 57.3, Gemini 3.1 Pro at 57.2)
- **Strongest gains over Qwen3.6:** CritPt +9.7 pts, Humanity's Last Exam +9.2 pts, Terminal-Bench Hard +6.9 pts
- **Long-horizon autonomy:** Alibaba reports sustained 1,000+ tool calls and 35-hour autonomous execution in internal testing (treat as vendor claim, not independently verified)
- **Designed for:** kernel optimization, code debugging, office workflow automation, multi-step data pipelines
- **API access:** DashScope (`dashscope-intl.aliyuncs.com`), both OpenAI and Anthropic API specs supported

### Wording behaviors that matter

- **Granularity beats inference** (family rule #1) — more sharply true on 3.7 than earlier versions
- **Self-verification prompts** boost completeness substantially
- **Higher abstention** — for fact-recall tasks, supply reference text
- **Skips un-emphasized sections** — number requirements explicitly

### When 3.7-Max specific tuning helps

- Long-running autonomous coding (the 35-hour claim is the actual feature being sold)
- Tasks where the +21 pp hallucination reduction is more valuable than the −7.6 pp accuracy
- Workflows that benefit from 1 M context **and** can validate retrieval quality on the task

### When NOT to invest in 3.7-Max specific tuning

- Visual / aesthetic design work — Qwen lags here
- Closed-book factual queries — high abstention will hurt
- Latency-critical interactive UIs — thinking mode is too slow
- Cost-sensitive batch workloads — DashScope pricing applies (no self-host)

### Migration from Qwen3.6 → Qwen3.7-Max

- Budget for higher abstention rates on fact-recall tasks
- Trim factual-recall-dependent phrasing
- Validate any long-context dependencies in your task (1 M nominal, but verify)
- Tests that worked on 3.6's 256 K context should still work; tests that depended on 3.6's particular phrasing may need granularity tightening

---

## Qwen3.7 Plus (Alibaba, May 2026)

Lower-cost sibling of Qwen3.7-Max. Same family character, less forgiving of vague prompts.

### When 3.7 Plus specific tuning helps

- High-volume routine tasks (customer support, content classification, predictable workloads)
- Cost-sensitive deployments where the Max premium isn't worth it
- Tasks where speed and efficiency outrank novelty

### Wording adjustments from 3.7-Max → 3.7 Plus

- **More granular spec needed** — Plus is more likely to skip sections than Max
- **Stronger self-verification** — add explicit "review your response" instructions
- **Fewer ambiguous open-endings** — "summarize this" works on Max, less reliably on Plus

---

## Qwen3.6 Plus (Alibaba, March 31 / April 2, 2026)

### Headline facts

- **1 M context window** (same as 3.7-Max)
- **Hybrid architecture:** linear attention + sparse MoE routing (linear attention is what makes 1 M context computationally tractable)
- **Available on OpenRouter** as a free preview when first released — broad accessibility

### Wording differences from 3.7-Max

- Slightly less abstention bias — recalls more facts but hallucinates somewhat more
- Polished output quality similar to 3.7-Max's; the 3.6 → 3.7 jump is on reasoning benchmarks more than UX polish

---

## Qwen3.6-Max-Preview (Alibaba, April 20, 2026)

### Headline facts

- **256 K context** (smaller than 3.6 Plus's 1 M — a deliberate distinction)
- **`preserve_thinking` feature** — carries reasoning traces across multi-turn conversations
- **More capable than 3.6 Plus** on the heaviest reasoning benchmarks at the time
- **Closed weights** (the original "closed flagship" before 3.7-Max replaced it)

### Wording-specific behavior

- The `preserve_thinking` flag pattern is analogous to Kimi's `keep: 'all'` — both vendors landed on similar semantics. Wording-side, you don't change the prompt; the flag is API config.

---

## Qwen3-Max-Thinking (Alibaba, January 25, 2026)

The original "thinking-mode" flagship. Mostly superseded by 3.6 / 3.7. Headline detail: scored 58.3 on HLE, 100% on AIME25, at roughly 12× lower input cost / 10× lower output cost than GPT-5.2-Thinking — the cost-efficiency story that drove Qwen adoption.

If you encounter a prompt tuned for Qwen3-Max-Thinking, it likely ports forward to 3.7-Max with no rewrites needed. Same family character.

---

## Cross-model wording principles (frontier Qwen side)

Mirror of the family-wide section, distilled for cross-model reviews:

- **Granularity > inference** — more explicit constraints than other vendors
- **Skips un-emphasized sections** — number requirements
- **Thinking mode toggle is the lever** — not prose
- **Self-verification prompts** are load-bearing on Qwen (free on other vendors)
- **Higher abstention** on factual recall — supply reference text
- **Iterative refinement** beats one-shot rewrites
- **OpenAI + Anthropic API compat** — wording is portable
- **1 M context unverified at scale** — measure before committing
- **Visual polish lags** — not the target for design / UI quality work

---

## Cross-vendor rules (when frontier Qwen is one of several targets)

If a prompt must run on Qwen **and** Claude / GPT / Gemini / Kimi / GLM:

| Axis | Direction on Qwen | Cross-vendor compromise |
|---|---|---|
| Granularity | helps (more than other vendors) | err on the side of explicit constraints — costs nothing on Claude / Kimi, helps GPT-5.5 partially, load-bearing on Qwen |
| Self-verification | helps strongly | add to cross-vendor prompts — free on others, load-bearing on Qwen |
| Numbered section list | helps strongly | use; safe on every vendor |
| Persona | OK (closed-weight reasoning model, no clear directional preference documented) | functional persona only; matches Claude / Kimi / Gemini direction, only conflict is GPT-5.5 |
| Thinking lever | toggle | parameter, not prose |
| Few-shot | helps when format strict | OK on Claude / Kimi / Gemini / Qwen; the conflict is GPT-5.5 reasoning |
| Reference text for facts | strongly recommended | universal pattern; works on every vendor |
| Long context | use cautiously | 1 M nominal but unverified; for cross-vendor prompts, target the most conservative window in scope |

Cross-vendor wording note: Qwen's wording defaults overlap heavily with Claude's — granular specs, explicit step naming, self-verification. The main divergence is the abstention/factual-recall axis, which is unique to Qwen.

---

## Source notes

Qwen3.7-Max was released too recently (May 20-21, 2026) for many independent prompting analyses to exist. The behaviors documented here come from:

- The MarkTechPost analysis (technical summary at release)
- Independent hands-on review by Atal Upadhyay (May 19, 2026 — covers 3.7 Plus and Max)
- Alibaba's release-day claims (treated as vendor claims, marked when used)
- The GitHub QwenLM repository (architecture facts)

Several axes are marked `?` or "unverified" — long-context behavior at scale, schema-strict JSON semantics, MCP tool description format specifics. Default to conservative wording on those axes until your stack can verify.

Small-local Qwen variants are explicitly covered in `small-local.md` and `matrix-small.md`. If a user mentions just "Qwen" without specifying tier, ask which one — the prompting advice diverges sharply (frontier Qwen tolerates principles in prose; small-local Qwen 2-3B regresses on the same wording).
