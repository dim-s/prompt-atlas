# Model-specific wording differences — Alibaba Qwen frontier family

What changes about how you should PHRASE prompts for **frontier-class Qwen models**: **Qwen3.8-Max** / **Qwen3.8-Flash** (August 2026) / **Qwen3.8-Omni-Flash** (September 2026), and the previous Qwen3.7-Max (May 2026). Qwen3.7 Plus, Qwen3.6 Plus / Max-Preview and Qwen3-Max-Thinking are in `archive/models/qwen-frontier.md`. Companion to `claude.md`, `gpt.md`, `gemini.md`, `kimi.md`, `glm.md`, `deepseek.md`. This reference stays focused on wording — not API parameters, context sizes, or pricing; a knob or a window size is named only where a prompt line or a prompt architecture would fight it.

**Retirements on Qwen Cloud** (deprecation notices, read 2026-10-31): `qwen-turbo`, `qwq-plus`, `qvq-max`, `qwen-vl-max`, `qwen-vl-plus` retired 10.10.2026; the Qwen3.5-Plus family (`qwen3.5-plus`, its dated snapshots, `qwen3.5-397b-a17b`) stops 19.01.2027 (end of maintenance 18.11.2026), replacement `qwen3.8-flash`. A prompt pinned to those ids is a migration case, not a tuning one.

**This file covers frontier Qwen only.** Small-local Qwen variants (Qwen3 2B / 4B / e2b / e4b that run on LM Studio, Ollama, llama.cpp, vLLM at consumer hardware tiers) are covered in `small-local.md § Qwen` — different prompting regime, different failure modes, different reference matrix (`matrix-small.md`). When a user names just "Qwen" without size or tier, ask which one before applying advice from either file.

Frontier Qwen runs via **QwenCloud** (the official hosted API, OpenAI- and DashScope-compatible — the newer host), the legacy **DashScope** surface, the Qwen Chat web UI, or through cross-tool routers via the OpenAI-compat surface. As of Qwen3.8-Max, Alibaba's flagship is **open-weights** again for the first time at Max class (`Qwen3.8-2.4T-A95B` on Hugging Face), while the hosted `qwen3.8-max` adds vision input and non-thinking support on top of the open model.

---

## Family-wide rules (apply to all current frontier Qwen versions)

These hold across Qwen3.7-Max → Qwen3.8-Max → Qwen3.8-Flash / Omni-Flash. Rules #2, #4, #5 and #7 come from hands-on reports on Qwen3.6 / 3.7 and rule #6 from Alibaba's Qwen3.7-Max numbers; none has been re-documented for 3.8 — they are kept because the mitigations are cheap.

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

Reported on Qwen3.7: the model occasionally **omitted entire sections** of a multi-section task if those sections weren't given explicit emphasis — more on 3.7 Plus (archived) than on 3.7-Max. Not re-documented for 3.8 (`?`); the mitigation below is cheap, so it stays the family default.

Wording mitigation:
- **Number every required section** in the request ("Section 1, Section 2, Section 3 — all must be present")
- **Add a self-verification step** ("Before responding, list each requested section and confirm coverage")
- **Use explicit emphasis sparingly but deliberately** for must-not-miss requirements

This is the opposite-direction of Claude 4.5+ where aggressive emphasis overtriggers; on Qwen it's the bare minimum to ensure attention.

### 3. Thinking: an effort knob on every 3.8 model — an off-switch everywhere except `Qwen3.8-2.4T-A95B`

Qwen3.7-Max was documented as requiring thinking mode for the quality numbers Alibaba publishes (toggle in the Qwen Chat UI and as an API parameter). **Qwen3.8 moved to an effort model.** On Qwen Cloud, for `qwen3.8-max`, `qwen3.8-flash` and their siblings: *"Options: `low`, `medium`, `xhigh`. Default: `xhigh`. Cannot be set together with thinking_budget — setting both will cause an error."* OpenAI-style values are mapped — *"`max` is mapped to xhigh, `high` is mapped to xhigh, `minimal` is mapped to low, and `none` is mapped to enable_thinking=False."*

| Surface | Thinking | Off-switch | `preserve_thinking` default |
|---|---|---|---|
| hosted `qwen3.8-max` | hybrid | `enable_thinking: false` | **on** |
| hosted `qwen3.8-flash` | *"hybrid, thinking enabled by default"* | `enable_thinking: false` | **off** |
| open `Qwen3.8-2.4T-A95B` | **thinking-only** | none | — |
| open `Qwen3.8-Flash-Next` | on by default | `enable_thinking: False` (chat template) | **on** |

The `/think` / `/no_think` soft switches are documented for open-source Qwen3 hybrid models and `qwen-plus-2025-04-28` — not for the 3.8 line.

**Wording implication:**
- Don't write "think step by step" — set the knob, not the prose (`reasoning_effort` / `enable_thinking`)
- A "don't think / answer immediately" line is **structurally unimplementable on `Qwen3.8-2.4T-A95B`** — same class of antipattern as Kimi / GLM-5.3 / Grok thinking-forced-on cases (`antipatterns.md` #38); on every other 3.8 surface it is inert prose where `enable_thinking` belongs
- When latency matters (interactive UI, autocomplete), set effort `low` or disable thinking on a hybrid surface and accept lower quality — but note the vendor's caution for agents (Flash-Next card): *"In multi-turn agentic tasks, lower reasoning effort does not always reduce overall task completion time. Although it may produce faster per-turn responses, it can also lead to insufficient analysis, more failures, and repeated retries, which may increase total latency and token consumption."*
- For agentic workflows: keep thinking on at the default effort, budget for latency and output tokens
- **Round-trip reasoning the documented way where `preserve_thinking` is on** — for `qwen3.8-max`: *"You must send back all historical reasoning_content in the reasoning_content field. **Do NOT concatenate reasoning_content into the content field.**"* A harness that folds reasoning into the visible text degrades the model (client concern, worth flagging)

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

Qwen3.8-Max is the **first open Max-class release** (`Qwen3.8-2.4T-A95B`, 2.4T total / 95B active, open weights on Hugging Face since ~2026-08-12), breaking Qwen3.7-Max's closed-weights posture. The hosted `qwen3.8-max` is the open model plus features (vision input, non-thinking mode, 1M context by default, built-in tools). Prompts targeting Qwen3.8-Max may now also run self-hosted on the open weights — check which surface the prompt is built for before assuming hosted-only economics.

---

## Qwen3.8-Max (Alibaba, GA August 3, 2026 — current frontier)

Announced July 19, 2026 as "Qwen3.8-Max-Preview" and officially released (**GA 03.08**; open weights on Hugging Face since 12.08). This closes the atlas's earlier lead (П10) on the Preview — the model is now confirmed by first-party sources.

### Headline facts

- **Open weights: `Qwen3.8-2.4T-A95B`** — 2.4T total / 95B active MoE, the first open Qwen Max-class release; open weights on Hugging Face (`Qwen/Qwen3.8-2.4T-A95B`)
- **Context:** 1,000,000 tokens (max input 991,808; 983,616 in thinking mode; max output 131,072). Open-weight variant: 262,144 native, extensible to 1,010,000
- **Hosting:** **QwenCloud** (the new official hosted API; OpenAI- and DashScope-compatible) — the atlas's older "Qwen frontier = DashScope only" front is updated
- **Thinking:** hybrid on the hosted model (`enable_thinking` toggle; non-thinking mode supported; vision input supported). Open-source weights are **thinking-only** (thinking cannot be disabled; text-only). `reasoning_effort` `low` / `medium` / **`xhigh`** (default `xhigh`); `thinking_budget` (not together with `reasoning_effort`); `preserve_thinking` (on by default on hosted `qwen3.8-max`) — family rule #3
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

## Qwen3.8-Flash (Alibaba, August 25–27, 2026 — cost-tier sibling)

Released as an open-weight, multimodal MoE with an **early preview of the Qwen4 architecture** (Alizila 27.08; ModelScope teaser 25.08; the card calls the open model an *"experimental preview of the architecture that will underpin Qwen4"*). Positioned against DeepSeek-V4-Flash at a fraction of the price. Hosted `qwen3.8-flash` is, per the card, *"the official version based on Qwen3.8-Flash-Next with more production features, e.g., 1M context length by default, official built-in tools."*

### Headline facts

- **Open weights: `Qwen/Qwen3.8-Flash-Next`** — 125B main + **51B N-gram embedding** + 4B MTP, **6B activated per token**; hybrid attention: Gated DeltaNet (GDN) + **Qwen Sparse Attention (QSA)** (micro-block-level indexer), Gated Residual (GR), Muon optimizer — the Qwen4-architecture preview
- **License:** `qwen-community-1.0` (custom — **NOT Apache**; check terms before commercial self-hosting)
- **Context:** **262,144 native, extensible to 1,000,000** on the open weights; hosted `qwen3.8-flash` defaults to **1M with official built-in tools** (QwenCloud) — the two surfaces differ more than an implementation detail: a prompt sized for hosted-1M breaks on a 262K self-host
- **Multimodal:** image-text input (vision encoder); open weights are `image-text-to-text`
- Powers QwenWork Standard mode (−75% tokens per task, ~2× generation speed per Alizila)
- **Benchmarks** (vendor card): SWE-bench Pro **62.5** (vs DeepSeek-V4-Flash 56.0 / Opus-4.6 53.4), DeepSWE 1.1 58.7, SWE-bench Multilingual 81.0; competitive with DeepSeek-V4-Flash and Claude-Opus-4.6 across agentic coding, long-horizon office work (CoWorkBench), tool use (Toolathlon Verified), MathVision, AndroidWorld, ERQA

### Wording behaviors that matter

- **Family rules #1–#6 apply** (granularity, numbered sections, self-verification, iterative refinement, abstention, visual-polish lag) — Flash is a cheaper family member, not a different regime
- **Thinking knobs — now documented, and the two surfaces differ** (family rule #3 table):
  - Open `Qwen3.8-Flash-Next` — the card's example, verbatim: `"enable_thinking": True,  # on by default`, `"preserve_thinking": True, # on by default`, `reasoning_effort="xhigh",  # xhigh by default; supported levels are xhigh, medium, and low`. Thinking is disabled with `chat_template_kwargs: {"enable_thinking": False}` (on Qwen Cloud pass `"enable_thinking": False` directly). Recommended sampling per mode — thinking: `temperature=1.0, top_p=0.95, top_k=20, min_p=0.0, presence_penalty=0.0`; instruct: `temperature=0.7, top_p=0.80, top_k=20, min_p=0.0, presence_penalty=1.5` — so a self-hosted harness that flips modes must flip sampling with it; the prompt body stays neutral.
  - Hosted `qwen3.8-flash` (Qwen Cloud API reference): same `reasoning_effort` ladder and default (`xhigh`), hybrid with thinking on by default — but **`preserve_thinking` defaults to `false`**, the opposite of the open card. A multi-turn agent ported from a self-hosted Flash-Next to the hosted model loses its reasoning history unless it sets `preserve_thinking: true` (client concern — flag it when a hosted Flash agent "forgets" its earlier analysis).
- **The surface split is the review-critical fact** (hosted 1M + tools + `preserve_thinking` off vs open 262K native + `preserve_thinking` on): establish which one the prompt targets before long-context, multi-turn or tool-heavy recommendations
- **Agentic coding is the use case the card targets** — SWE-bench Pro 62.5 at this price tier puts it in the same review lane as DeepSeek V4.1 Flash (tight scope, explicit success criteria, tool descriptions matter)

### When Qwen3.8-Flash specific tuning helps

- Cost-sensitive coding / tool-driven agents at the Flash price tier (the named replacement for the retiring Qwen3.5-Plus family)
- Multimodal + high-volume workloads on the hosted 1M surface

### When NOT to invest

- Long-context reasoning on the **open weights** (262K native) where the hosted model's 1M is required

---

## Qwen3.8-Omni-Flash (Alibaba, September 18, 2026 — omni-input, API-only)

`qwen3.8-omni-flash`: natively omni **input** (text, images, audio, video) with **text-only output**; API-only at launch — no open weights, so the Flash surface split above does not apply here. Launch facts are second-hand (MarkTechPost, 2026-09-18, quoting the QwenCloud pages; the qwen.ai blog would not render). Read first-hand on Qwen Cloud (2026-10-31): the thinking guide lists it as *"hybrid, thinking enabled by default"* with *"Adjust thinking effort with `reasoning_effort`"*, and the API reference marks `preserve_thinking` *"(enabled by default)"* for it. Its effort ladder is not on the API reference's 3.8 list — still secondary.

### Headline facts

- **Context:** 1M window; QwenCloud lists 991K max input and 131K max output; reasoning up to 262K tokens
- **Thinking is on by default, with `reasoning_effort` set to `xhigh`; setting it to `none` disables thinking** (as quoted by MarkTechPost — the on-by-default part is now confirmed first-hand, the ladder is not). Qwen3.8-Flash's own knobs are documented separately (§ Qwen3.8-Flash) — don't infer one from the other
- Available through QwenCloud, Alibaba Cloud Model Studio and Qwen Studio

### Wording behaviors

- **Family rules #1–#6 apply**; no prompting guide was found and no wording delta versus Qwen3.8-Flash is documented. The difference for a reviewer is the input surface: audio/video prompts should name which clip or segment a question is about (same rule as naming images on DeepSeek Vision — `?` whether Qwen documents it).
- **Default `xhigh` is a cost/latency fact, not a wording one:** a latency-sensitive prompt is fixed with the parameter (`reasoning_effort`), not with a "don't overthink" line — same rule as every vendor with a knob.

### When NOT to invest

- Text-only workloads — use Qwen3.8-Flash or Max; nothing here changes the text regime.

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

### Migration from Qwen3.6 (archived) → Qwen3.7-Max

- Budget for higher abstention rates on fact-recall tasks
- Trim factual-recall-dependent phrasing
- Validate any long-context dependencies in your task (1 M nominal, but verify)
- Tests that worked on 3.6's 256 K context should still work; tests that depended on 3.6's particular phrasing may need granularity tightening

---

## Cross-model wording principles (frontier Qwen side)

Mirror of the family-wide section, distilled for cross-model reviews:

- **Granularity > inference** — more explicit constraints than other vendors
- **Skips un-emphasized sections** — number requirements
- **Thinking knobs are the lever, not prose** — `reasoning_effort` (low/medium/xhigh, default xhigh) + `enable_thinking` on 3.8 (no off-switch only on `Qwen3.8-2.4T-A95B`); toggle on 3.7-Max; `preserve_thinking` default differs by surface (hosted Flash off, hosted Max and open Flash-Next on)
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
| Thinking lever | **effort model on 3.8** (`reasoning_effort` low/medium/xhigh, default xhigh; `enable_thinking` on hosted Max / Flash and open Flash-Next; thinking-only on open `Qwen3.8-2.4T-A95B`) | parameter, not prose |
| Few-shot | helps when format strict | OK on Claude / Kimi / Gemini / Qwen; the conflict is GPT-5.5 reasoning |
| Reference text for facts | strongly recommended | universal pattern; works on every vendor |
| Long context | 1M documented on 3.8-Max (input 991,808); 1M nominal-unverified on 3.7-Max | for cross-vendor prompts, target the most conservative window in scope |

Cross-vendor wording note: Qwen's wording defaults overlap heavily with Claude's — granular specs, explicit step naming, self-verification. The main divergence is the abstention/factual-recall axis, which is unique to Qwen.

---

## Source notes

- Qwen3.8-Omni-Flash: MarkTechPost ([marktechpost.com/2026/09/18/alibaba-qwen-releases-qwen3-8-omni-flash](https://www.marktechpost.com/2026/09/18/alibaba-qwen-releases-qwen3-8-omni-flash/), read 2026-09-24) — modalities, 991K/131K limits, `reasoning_effort` default `xhigh`, `none` disables; the primary `qwen.ai/blog?id=qwen3.8-omni-flash` did not render (secondary source)
- Read 2026-10-31 (raw): Hugging Face card [huggingface.co/Qwen/Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next/raw/main/README.md) — `enable_thinking` / `preserve_thinking` / `reasoning_effort` defaults, per-mode sampling, the agentic low-effort caution, the Qwen4-preview wording; Qwen Cloud API reference [docs.qwencloud.com/api-reference/chat/openai-chat](https://docs.qwencloud.com/api-reference/chat/openai-chat.md) — the 3.8 `reasoning_effort` ladder and mappings, `preserve_thinking` default `false` (on by default only for `qwen3.8-max`, `-0902`, `qwen3.8-omni-flash`), the do-not-concatenate rule; thinking guide [docs.qwencloud.com/developer-guides/text-generation/thinking](https://docs.qwencloud.com/developer-guides/text-generation/thinking.md) — hybrid vs thinking-only per model, `/think` `/no_think` scope; deprecation notices [docs.qwencloud.com/changelog/model-deprecation](https://docs.qwencloud.com/changelog/model-deprecation.md) and its Qwen3.5 legacy / historical-mainline pages — the 10.10 retirements and the 19.01.2027 Qwen3.5-Plus date (the "mainline" and "snapshot" notices are images and were not read)
- Qwen3.8-Flash: Alizila (27.08, [alizila.com](https://www.alizila.com/alibaba-releases-qwen3-8-flash-with-innovative-model-architecture-delivering-optimal-price-performance/)) — 125B+51B/6B-active specs, price tiers (¥1/¥3), QwenWork, benchmark positioning; Hugging Face card `Qwen/Qwen3.8-Flash-Next` (read 2026-08-29) — architecture details (GDN/QSA/GR, N-gram embedding, Muon), context numbers (262K/1M), license `qwen-community-1.0`, SWE-bench comparisons; QwenCloud model page for hosted `qwen3.8-flash` (1M default, built-in tools). Thinking knobs were not documented as of 29.08; closed by the 2026-10-31 reads above

Qwen3.7-Max was released too recently (May 20-21, 2026) for many independent prompting analyses to exist. Qwen3.8-Max facts come from the vendor's own pages:

- Alibaba Cloud Model Studio model page ([alibabacloud.com/help/en/model-studio/qwen3-8-max](https://www.alibabacloud.com/help/en/model-studio/qwen3-8-max)) — context limits (1,000,000 window; input 991,808 / 983,616 thinking; output 131,072) and QwenCloud pricing ($2/$6, implicit cache $0.25 on the international surface)
- Qwen team blog ([qwen.ai/blog?id=qwen3.8](https://qwen.ai/blog?id=qwen3.8)) and the team's Hugging Face card for `Qwen3.8-2.4T-A95B` — 2.4T total / 95B active, open Max-class release, reasoning_effort (xhigh default), preserve_thinking on by default, open-source variant requires thinking mode (cannot be disabled), self-host options (SGLang / vLLM / TokenSpeed)
- QwenCloud developer guide ([docs.qwencloud.com/developer-guides/text-generation/thinking](https://docs.qwencloud.com/developer-guides/text-generation/thinking)) — hybrid vs thinking-only modes, `enable_thinking` / `thinking_budget` / `/think` `/no_think` semantics
- The GA date (03.08) and Max-Preview→Max closure are per the release coverage cited in the atlas briefing (CGTN, 03.08); the open-weights date (12.08) is from the HF organisation

Several axes are marked `?` or "unverified" — long-context behavior of the open weights at scale, schema-strict JSON semantics, MCP tool description format specifics. Default to conservative wording on those axes until your stack can verify.

Small-local Qwen variants are explicitly covered in `small-local.md` and `matrix-small.md`. If a user mentions just "Qwen" without specifying tier, ask which one — the prompting advice diverges sharply (frontier Qwen tolerates principles in prose; small-local Qwen 2-3B regresses on the same wording).
