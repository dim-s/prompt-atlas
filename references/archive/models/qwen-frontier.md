# Archive — legacy sections from `models/qwen-frontier.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

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

## Qwen3.6 Plus (Alibaba, March 31 / April 2, 2026)

### Headline facts

- **1 M context window** (same as 3.7-Max)
- **Hybrid architecture:** linear attention + sparse MoE routing (linear attention is what makes 1 M context computationally tractable)
- **Available on OpenRouter** as a free preview when first released — broad accessibility

### Wording differences from 3.7-Max

- Slightly less abstention bias — recalls more facts but hallucinates somewhat more
- Polished output quality similar to 3.7-Max's; the 3.6 → 3.7 jump is on reasoning benchmarks more than UX polish

## Qwen3.6-Max-Preview (Alibaba, April 20, 2026)

### Headline facts

- **256 K context** (smaller than 3.6 Plus's 1 M — a deliberate distinction)
- **`preserve_thinking` feature** — carries reasoning traces across multi-turn conversations
- **More capable than 3.6 Plus** on the heaviest reasoning benchmarks at the time
- **Closed weights** (the original "closed flagship" before 3.7-Max replaced it)

### Wording-specific behavior

- The `preserve_thinking` flag pattern is analogous to Kimi's `keep: 'all'` — both vendors landed on similar semantics. Wording-side, you don't change the prompt; the flag is API config.

## Qwen3-Max-Thinking (Alibaba, January 25, 2026)

The original "thinking-mode" flagship. Mostly superseded by 3.6 / 3.7. Headline detail: scored 58.3 on HLE, 100% on AIME25, at roughly 12× lower input cost / 10× lower output cost than GPT-5.2-Thinking — the cost-efficiency story that drove Qwen adoption.

If you encounter a prompt tuned for Qwen3-Max-Thinking, it likely ports forward to 3.7-Max with no rewrites needed. Same family character.
