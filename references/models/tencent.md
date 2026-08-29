# Model-specific wording differences — Tencent Hunyuan family

What changes about how you should PHRASE prompts for **Tencent Hy4 preview** (Tencent Hunyuan, open weights, 2026-08-28) — the first Tencent family in this atlas. Companion to the other model files in this directory.

Coverage is deliberately short and marked with `?`. Tencent has published **no prompting guide** for Hy4: the facts below come from the vendor's launch release and availability surfaces, and everything beyond them stays `?`. That is the same treatment as Mistral Medium 3.5 and the Muse family — facts of availability are recorded, behavioral guesses are not.

---

## Family-wide notes

### 1. Preview-first release posture — treat the model as moving

Hy4 preview is the first step of a "preview-first approach, followed by official releases" (vendor wording; the next batch of the Hy4 series is expected soon). Prompts written today may be tuned against a moving target: prefer stable, cross-vendor-safe wording and mark Hy4-specific assumptions as re-testable.

### 2. No prompting guide — apply universal defaults, don't invent

None of the behavioral axes (persona, few-shot, literalism, emphasis, instruction placement, reasoning knobs) are documented by the vendor. When a review targets Hy4: apply the cross-vendor universal defaults (`_universal.md`), keep load-bearing instruction placement conventional (bulk in system prompt is the universal default), and mark every version-specific cell `?`. A review that claims a Hy4-specific behavioral delta beyond the facts below is fabricating.

---

## Tencent Hy4 preview (August 28, 2026)

### Headline facts

- **Architecture:** 770 B total / **49 B active** per token (MoE), open-source
- **Context:** **>1M tokens** (vendor claim; usable-window behavior unverified)
- **Availability:** open weights; API via **Tencent Cloud TokenHub** and **OpenRouter**; consumer/product surfaces: WorkBuddy, CodeBuddy, Yuanbao, ima. Free on WorkBuddy / CodeBuddy for two weeks at launch (Hy3 free access extended to 30.09)
- **Pricing (API):** $0.834 / M input, $2.501 / M output, **$0.042 / M cache hit**
- **Positioning:** real-world productivity — coding, office work (documents/spreadsheets/presentations), scientific research; game-dev prototype from a single natural-language request (vendor claim); recursive self-improvement loop claimed (model contributed to its own training/inference optimization, +31.8% inference throughput — vendor-reported, not independently verified)
- **Blind internal eval:** 2.99 / 4.00 vs GLM-5.3 2.92 and Kimi K3 2.94 (163 experts, 203 engineering tasks — vendor's own panel)

### What is `?`

- Reasoning/thinking surface: **no documented knob** (`reasoning_effort`-class parameter unknown for both TokenHub and OpenRouter surfaces) — do not claim one; ask the user what their endpoint passes
- Persona response, few-shot behavior, literalism, emphasis, verbosity default, instruction-placement sensitivity, tool-calling format quirks, long-context degradation curve

### Wording behaviors worth stating (facts, not guesses)

- **Long-context claims need validation per surface** — >1M is a claim; size prompts against what the actual endpoint holds in practice (same discipline as GLM-5.3-Flash's 1M-vs-300K and Qwen3.8 open-weights 262K cases)
- **Cached-input economics are the agent-relevant number** ($0.042/M cache hits) — a stable system block re-sent every step is cheap, like DeepSeek's DSA rule but unconfirmed at scale; prefer stability in the persistent part only after measuring cache behavior on the endpoint
- **Productivity / agentic framing matches Chinese-ecosystem conventions** — outcome-defined success criteria and explicit verification loops are safe wording anywhere and cost nothing

### When Hy4 specific tuning helps

- Cost-sensitive coding / office-automation agents routed through OpenRouter / TokenHub where the open-weight economics beat closed frontiers
- Prompts that must also run on other OpenRouter-hosted open models — Hy4 as a member of the open-weight tier (GLM-5.3-Flash, Qwen3.8-Flash, DeepSeek V4-Flash) shares that lane's conventions: explicit scope, tool descriptions, schema-driven output

### When NOT to invest

- Until the vendor publishes prompting guidance: any claim of Hy4-specific wording deltas
- Workloads that depend on a documented reasoning-effort knob — none exists yet

---

## Cross-vendor rules (when Hy4 is one of several targets)

| Axis | Direction on Tencent Hy4 | Cross-vendor alignment |
|---|---|---|
| Instruction placement | `?` (no guidance) | use the universal default (bulk in system) unless another vendor in the set forces otherwise |
| Reasoning depth | `?` — no documented knob | never embed "think harder" prose; surface the knob question to the user |
| Context sizing | >1M claimed, unverified | target the most conservative verified window in the set |
| Cached-input economics | $0.042/M claimed | safe everywhere; the reward is real only if the endpoint caches (verify) |
| Persona / few-shot / emphasis / literalism | `?` | let the other vendors in the set decide these cells |

---

## Source notes

- Tencent's launch release, [Tencent Releases and Open-Sources Tencent Hy4 preview](https://www.tencent.com/tencent-releases-and-open-sources-tencent-hy4-preview/) (2026-08-28, read 2026-08-29) — 770B/49B, >1M context, open weights, TokenHub / OpenRouter availability, product surfaces (WorkBuddy, CodeBuddy, Yuanbao, ima), pricing ($0.834/$2.501/$0.042), the blind-eval 2.99 vs GLM-5.3 2.92 / Kimi K3 2.94, the preview-first roadmap, self-improvement claims (vendor-reported)
- Superpower Daily / secondary coverage (28.08) — corroboration of the architecture numbers and the open-source framing
- **No prompting guide published as of 2026-08-29** — every unlisted behavioral axis is deliberately `?`
