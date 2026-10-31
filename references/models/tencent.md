# Model-specific wording differences — Tencent Hunyuan family

What changes about how you should PHRASE prompts for **Tencent Hy4 preview** (Tencent Hunyuan, open weights, 2026-08-28). Companion to the other model files in this directory. This reference stays focused on wording — not API parameters, context sizes, or pricing, except where one changes prompt design.

Coverage is deliberately short and marked with `?`. Tencent has published **no prompting guide** for Hy4: the facts below come from the launch release, the Hugging Face card and its chat template, and everything beyond them stays `?`. Same treatment as Mistral Medium 3.5 and the Muse family — facts are recorded, behavioral guesses are not.

**Not prompt targets:** Tencent's October 2026 Hy releases are specialised components, not general chat models — `Hy-Memory-30B-A3B` (15.10, *"a compact long-term-memory writer model"* that replaces the LLM inside memory frameworks), `Hy-SimulMT-30B-A3B` (22.10, simultaneous speech translation), `Hy-Nomas` (21.10, *"a video world generation model"*). No sections for them.

---

## Family-wide notes

### 1. Preview-first release posture — treat the model as moving

Hy4 preview is the first step of a "preview-first approach, followed by official releases" (vendor wording). No Hy4 GA had shipped as of 2026-10-31 (hy.tencent.com newest item 2026-09-21; the HF `Hy4-preview` repo unchanged since 08-27/28). Prefer stable, cross-vendor-safe wording and mark Hy4-specific assumptions as re-testable.

### 2. No prompting guide — apply universal defaults, don't invent

Persona, few-shot, literalism, emphasis and instruction-placement behavior are undocumented. When a review targets Hy4: apply the cross-vendor universal defaults (`_universal.md`), keep instruction placement conventional (bulk in system prompt is the universal default), and mark version-specific cells `?`. The reasoning knob **is** documented for the open weights (below); for the hosted TokenHub surface it is not verified.

---

## Tencent Hy4 preview (August 28, 2026)

### Headline facts

- **Architecture:** 770B total / **49B active** per token (MoE), open weights, Apache-2.0
- **Context:** 1M (card); usable-window behavior unverified
- **Availability:** open weights; API via **Tencent Cloud TokenHub** and **OpenRouter**; product surfaces WorkBuddy, CodeBuddy, Yuanbao, ima
- **Reasoning knob (open weights, card + template):** `reasoning_effort` in `chat_template_kwargs`, only **`high`** (default) or **`no_think`** — *"Defaults to `"high"` (deep chain-of-thought), which suits complex tasks such as math, coding, and reasoning. For direct responses, pass `extra_body={"chat_template_kwargs": {"reasoning_effort": "no_think"}}`."* Any other value raises *"should be no_think/high"*. So there is an on/off switch, not a ladder.
- **Sampling (card):** *"Recommended parameters: `temperature=0.9`, `top_p=1.0`."*
- **Template mechanics:** the reasoning mode is rendered into the **leading system message** (`reasoning_effort:{value}`); with no system message the template synthesizes one. `preserved_thinking` defaults to false without tools and true with tools; *"no_think overrides preserved_thinking: history is always emitted with empty think tags"*.
- **Known issues, stated by the vendor:** *"we are shipping with known issues — among them, spending longer than necessary reasoning through complex tasks, and a tendency to over-verify its own work."*
- **Positioning:** coding, office work (documents / spreadsheets / presentations), scientific research (vendor claims)

### What is `?`

- Reasoning knob on **TokenHub** and **OpenRouter** — the card documents the self-hosted template only; ask the user what their endpoint passes
- Persona response, few-shot behavior, literalism, emphasis, verbosity default, instruction-placement sensitivity, tool-calling format quirks, long-context degradation curve

### Wording behaviors worth stating (from the facts above)

- **Don't add self-verification scaffolding.** The vendor names over-verification as a known issue; "double-check your work" / "verify before finishing" lines stack on it. Keep task checks (run the tests, check against tool results) and drop the ritual — same split as the Opus 5 lineage in `claude.md`.
- **Reasoning depth is binary — don't prose-steer it.** "Think briefly" or "answer directly" in the body does nothing the template doesn't already offer; for simple turns, `no_think` is the lever. Overlong reasoning on complex tasks is a known issue with no documented intermediate level.
- **A system message is always present.** The template writes the reasoning mode into the first system message (or creates one), so "no system prompt" deployments are not truly system-free — review any identity or behavior assumptions with that in mind.
- **Tool-less multi-turn chats drop prior reasoning** (`preserved_thinking` false) — don't ask the model to build on its earlier-turn reasoning; restate the conclusion in a visible turn.
- **Cached input is the agent-relevant economics** — the launch release prices cache hits at ~1/20 of uncached input; a byte-stable system block re-sent every step is cheap if the endpoint caches (verify per surface).
- **Long-context claims need validation per surface** — size prompts against what the actual endpoint holds.

### When Hy4 specific tuning helps

- Cost-sensitive coding / office-automation agents on OpenRouter / TokenHub / self-host where open-weight economics beat closed frontiers
- Prompts that must also run on other open-weight models (GLM-5.3-Flash, Qwen3.8-Flash, DeepSeek V4.1 Flash) — shared lane conventions: explicit scope, tool descriptions, schema-driven output

### When NOT to invest

- Any claim of Hy4-specific wording deltas beyond the facts above, until the vendor publishes prompting guidance
- Workloads that need a graded reasoning-effort ladder — Hy4 preview offers `high` or `no_think` only

---

## Cross-vendor rules (when Hy4 is one of several targets)

| Axis | Direction on Tencent Hy4 preview | Cross-vendor alignment |
|---|---|---|
| Instruction placement | `?` (no guidance) | use the universal default (bulk in system) unless another vendor in the set forces otherwise |
| Reasoning depth | `reasoning_effort` `high` (default) / `no_think` on open weights; hosted `?` | never embed "think harder / less" prose; surface the knob to the user |
| Self-verification | vendor-named over-verification issue | phrase verification as part of the task, not a self-review pass |
| Context sizing | 1M (card), unverified in use | target the most conservative verified window in the set |
| Cached-input economics | cache hit ~1/20 of input (launch release) | safe everywhere; real only if the endpoint caches (verify) |
| Persona / few-shot / emphasis / literalism | `?` | let the other vendors in the set decide these cells |

---

## Source notes

- HF [tencent/Hy4-preview card](https://huggingface.co/tencent/Hy4-preview/raw/main/README.md) — read 2026-10-31 (raw): Apache-2.0, context 1M, recommended `temperature=0.9` / `top_p=1.0`, reasoning mode `high` default / `no_think`, the known-issues paragraph (over-long reasoning, over-verification)
- HF [Hy4-preview chat_template.jinja](https://huggingface.co/tencent/Hy4-preview/raw/main/chat_template.jinja) — read 2026-10-31 (raw): `reasoning_effort` only `high` / `no_think`, `preserved_thinking` default by tools presence, reasoning token in the leading system message, synthesized system message, `no_think` empties history think tags
- HF cards [Hy-Memory-30B-A3B](https://huggingface.co/tencent/Hy-Memory-30B-A3B), [Hy-SimulMT-30B-A3B](https://huggingface.co/tencent/Hy-SimulMT-30B-A3B), [Hy-Nomas](https://huggingface.co/tencent/Hy-Nomas) and the HF API list for `tencent` — read 2026-10-31 (raw): specialised models, dates 15.10 / 22.10 / 21.10; Hy4-preview repo created 2026-08-27
- hy.tencent.com news list — read 2026-10-31 (headless render): newest item 2026-09-21, no Hy4 GA
- Tencent's launch release, [Tencent Releases and Open-Sources Tencent Hy4 preview](https://www.tencent.com/tencent-releases-and-open-sources-tencent-hy4-preview/) (2026-08-28, read 2026-08-29) — 770B/49B, TokenHub / OpenRouter availability, product surfaces, cache-hit vs input pricing (ratio used above), preview-first roadmap
- Superpower Daily / secondary coverage (28.08) — corroboration of the architecture numbers
- **Not read:** Tencent Cloud TokenHub docs (hosted Hy4 knobs) — hence the hosted `?`
- **No prompting guide published as of 2026-10-31** (card, template and launch release carry none) — every unlisted behavioral axis is deliberately `?`
