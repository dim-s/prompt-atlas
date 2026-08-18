# Model-specific wording differences — Alibaba Qwen frontier family

What changes about how you should PHRASE prompts for **frontier-class Qwen models**: Qwen3.8-Max (August 2026), Qwen3.7-Max (May 2026), Qwen3.7 Plus, Qwen3.6 Plus / Max-Preview, and the Qwen3-Max-Thinking lineage. Companion to `claude.md`, `gpt.md`, `gemini.md`, `kimi.md`, `glm.md`, `deepseek.md`.

**This file covers frontier Qwen only.** Small-local Qwen variants (Qwen3 2B / 4B / e2b / e4b that run on LM Studio, Ollama, llama.cpp, vLLM at consumer hardware tiers) are covered in `small-local.md § Qwen` — different prompting regime, different failure modes, different reference matrix (`matrix-small.md`). When a user names just "Qwen" without size or tier, ask which one before applying advice from either file.

Frontier Qwen runs via **QwenCloud** (the official hosted API, OpenAI- and DashScope-compatible — the newer host), the legacy **DashScope** surface, the Qwen Chat web UI, or through cross-tool routers via the OpenAI-compat surface. As of Qwen3.8-Max, Alibaba's flagship is **open-weights** again for the first time at Max class (`Qwen3.8-2.4T-A95B` on Hugging Face), while the hosted `qwen3.8-max` adds vision input and non-thinking support on top of the open model.

---

## Family-wide rules (apply to all current frontier Qwen versions)

These hold across Qwen3.6 Plus → Qwen3.6-Max-Preview → Qwen3-Max-Thinking → Qwen3.7-Max → Qwen3.8-Max.

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

### 3. Thinking mode is the quality lever — with knobs, and (on open weights) no off-switch

Qwen3.7 Max / Plus are documented as "requiring thinking mode enabled" for the quality numbers Alibaba publishes; the toggle is exposed in the Qwen Chat UI and as an API parameter. **Qwen3.8 changed the parameter surface**: reasoning depth is now an effort model with `reasoning_effort` (`low` / `medium` / **`xhigh`**, default `xhigh`), plus `thinking_budget` (token cap for the CoT) and `preserve_thinking` (carry reasoning across turns — enabled by default on Qwen3.8).

Two surfaces differ:
- **Hosted `qwen3.8-max`** (QwenCloud): hybrid — `enable_thinking` toggles thinking per request; vision input and non-thinking mode supported.
- **Open-source `Qwen3.8-2.4T-A95B`** (HF): **thinking-only** — requires thinking mode for all interactions, cannot be disabled; multimodal input not supported. Hybrid soft-switches `/think` / `/no_think` are available for open-source Qwen3 hybrid models (last instruction wins), and `enable_thinking` is on by default.

**Wording implication:**
- Don't write "think step by step" — set the knob, not the prose (`reasoning_effort` / `enable_thinking`)
- A "don't think / answer immediately" line is **structurally unimplementable on the open-source Qwen3.8 weights** — same class of antipattern as Kimi / GLM-5.3 / Grok thinking-forced-on cases (`antipatterns.md` #38)
- When latency matters (interactive UI, autocomplete), set effort `low` or disable thinking on the hosted model and accept lower quality
- For agentic workflows: enable thinking / raise effort, budget for latency and output tokens (thinking tokens bill at output rates)

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

### 9. Long-context: 1M is real on Qwen3.8-Max, unverified on Qwen3.7-Max

Qwen3.8-Max's 1M context is **documented with concrete limits**: context window 1,000,000, max input 991,808 (983,616 in thinking mode), max output 131,072 on the hosted model; the open weights are 262,144 native, extensible to 1,010,000. Qwen3.7-Max's advertised 1M was *not* independently verified at the time.

**Wording rule:** the hosted Qwen3.8-Max's long-context numbers are vendor-documented and usable; the open-source weights' full-window behavior should still be validated against your task before committing long-context prompt patterns.

### 10. Hosted flagship is open-weights at Max class for the first time

Qwen3.8-Max is the **first open Max-class release** (`Qwen3.8-2.4T-A95B`, 2.4T total / 95B active, open weights on Hugging Face since ~2026-08-12), breaking Qwen3.7-Max's closed-weights posture. The hosted `qwen3.8-max` is the open model plus features (vision input, non-thinking mode, 1M context by default, built-in tools). Pricing is QwenCloud/dashscope rates: **$2 / $6 per MTok input/output with implicit cache $0.25** on the international (Singapore) surface. Prompts targeting Qwen3.8-Max may now also run self-hosted on the open weights — check which surface the prompt is built for before assuming hosted-only economics.

---

## Qwen3.8-Max (Alibaba, GA August 3, 2026 — current frontier)

Announced July 19, 2026 as "Qwen3.8-Max-Preview" and officially released (**GA 03.08**; open weights on Hugging Face since 12.08). This closes the atlas's earlier lead (П10) on the Preview — the model is now confirmed by first-party sources.

### Headline facts

- **Open weights: `Qwen3.8-2.4T-A95B`** — 2.4T total / 95B active MoE, the first open Qwen Max-class release; open weights on Hugging Face (`Qwen/Qwen3.8-2.4T-A95B`)
- **Context:** 1,000,000 tokens (max input 991,808; 983,616 in thinking mode; max output 131,072). Open-weight variant: 262,144 native, extensible to 1,010,000
- **Cost:** $2 / $6 per MTok (input/output), implicit cache $0.25 (international/QwenCloud surface)
- **Hosting:** **QwenCloud** (the new official hosted API; OpenAI- and DashScope-compatible) — the atlas's older "Qwen frontier = DashScope only" front is updated
- **Thinking:** hybrid on the hosted model (`enable_thinking` toggle; non-thinking mode supported; vision input supported). Open-source weights are **thinking-only** (thinking cannot be disabled; text-only). `reasoning_effort` `low` / `medium` / **`xhigh`** (default `xhigh`); `thinking_budget`; `preserve_thinking` (on by default); `/think` / `/no_think` soft switches for open-source hybrid models
- **Positioning:** Alibaba reports autonomous end-to-end project work ("code for over ten days" is the vendor's framing), native visual understanding across planning/execution/verification on the hosted model; strong coding + cowork results (PaperBench 93.0, Terminal-Bench 2.1 86.6, QwenSWEBench 80.7 per the vendor's card)

### Wording behaviors that matter

- Family rules #1–#5 apply unchanged (granularity, numbered sections, self-verification, iterative refinement)
- The thinking lever is **parameter-based** (`reasoning_effort` / `enable_thinking`), never prose — and on the open weights a "don't think" instruction is unimplementable
- Self-verification prompts remain load-bearing (family rule #5), consistent with the 3.7 line

### Migration from Qwen3.7-Max → Qwen3.8-Max

- 3.7-era prompts run forward-compatibly; the main action is moving reasoning control from the old toggle/required-on assumption to the effort model (`reasoning_effort`, default `xhigh`)
- Budget for the higher default effort: `xhigh` on 3.8 costs more thinking tokens than 3.7's required-on mode at its default
- If the target is the self-hosted open weights, re-check multimodal assumptions — the open model is text-only

---

## Qwen3.7-Max (Alibaba, May 20-21, 2026 — previous frontier)

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
- **1 M context real on 3.8-Max (documented limits), unverified on 3.7-Max** — measure before committing
- **Visual polish lags** — not the target for design / UI quality work

---

## Cross-vendor rules (when frontier Qwen is one of several targets)

If a prompt must run on Qwen **and** Claude / GPT / Gemini / Kimi / GLM:

| Axis | Direction on Qwen | Cross-vendor compromise |
|---|---|---|
| Granularity | helps (more than other vendors) | err on the side of explicit constraints — costs nothing on Claude / Kimi, helps GPT-5.5 partially, load-bearing on Qwen |
| Self-verification | helps strongly | add to cross-vendor prompts — free on others, load-bearing on Qwen |
| Numbered section list | helps strongly | use; safe on every vendor |
| Persona | OK (reasoning model, no clear directional preference documented) | functional persona only; matches Claude / Kimi / Gemini direction, only conflict is GPT-5.5 |
| Thinking lever | **effort model on 3.8-Max** (`reasoning_effort` low/medium/xhigh, default xhigh; `enable_thinking` on hosted; thinking-only on open weights) | parameter, not prose |
| Few-shot | helps when format strict | OK on Claude / Kimi / Gemini / Qwen; the conflict is GPT-5.5 reasoning |
| Reference text for facts | strongly recommended | universal pattern; works on every vendor |
| Long context | 1M documented on 3.8-Max (input 991,808); 1M nominal-unverified on 3.7-Max | for cross-vendor prompts, target the most conservative window in scope |

Cross-vendor wording note: Qwen's wording defaults overlap heavily with Claude's — granular specs, explicit step naming, self-verification. The main divergence is the abstention/factual-recall axis, which is unique to Qwen.

---

## Source notes

Qwen3.7-Max was released too recently (May 20-21, 2026) for many independent prompting analyses to exist. Qwen3.8-Max facts come from the vendor's own pages:

- Alibaba Cloud Model Studio model page ([alibabacloud.com/help/en/model-studio/qwen3-8-max](https://www.alibabacloud.com/help/en/model-studio/qwen3-8-max)) — context limits (1,000,000 window; input 991,808 / 983,616 thinking; output 131,072) and QwenCloud pricing ($2/$6, implicit cache $0.25 on the international surface)
- Qwen team blog ([qwen.ai/blog?id=qwen3.8](https://qwen.ai/blog?id=qwen3.8)) and the team's Hugging Face card for `Qwen3.8-2.4T-A95B` — 2.4T total / 95B active, open Max-class release, reasoning_effort (xhigh default), preserve_thinking on by default, open-source variant requires thinking mode (cannot be disabled), self-host options (SGLang / vLLM / TokenSpeed)
- QwenCloud developer guide ([docs.qwencloud.com/developer-guides/text-generation/thinking](https://docs.qwencloud.com/developer-guides/text-generation/thinking)) — hybrid vs thinking-only modes, `enable_thinking` / `thinking_budget` / `/think` `/no_think` semantics
- The GA date (03.08) and Max-Preview→Max closure are per the release coverage cited in the atlas briefing (CGTN, 03.08); the open-weights date (12.08) is from the HF organisation

Several axes are marked `?` or "unverified" — long-context behavior of the open weights at scale, schema-strict JSON semantics, MCP tool description format specifics. Default to conservative wording on those axes until your stack can verify.

Small-local Qwen variants are explicitly covered in `small-local.md` and `matrix-small.md`. If a user mentions just "Qwen" without specifying tier, ask which one — the prompting advice diverges sharply (frontier Qwen tolerates principles in prose; small-local Qwen 2-3B regresses on the same wording).
