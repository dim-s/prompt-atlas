# Archive — legacy sections from `models/grok.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

## Grok 4.5 (July 2026 — previous recommended model)

xAI's models page previously named it the default choice; it's superseded by Grok 4.6 (August 2026) and then Grok 4.7 (September 2026). Secondary sources put its release at July 8, 2026.

### Headline facts

- **500K context** — down from Grok 4.3's 1M
- **Knowledge cutoff:** February 1, 2026
- **Two-step pricing by prompt length:** $2.00 / $6.00 per M tokens (in / out) below 200K tokens of prompt, **doubling to $4.00 / $12.00 at 200K and above**
- **`reasoning_effort`: `low` / `medium` / `high` (default)** — `xhigh` is sent to "high" (not supported on 4.5); **reasoning cannot be disabled**

### The migration trap: newer model, smaller window

This is the rare case where an upgrade **narrows** a capability, and it fails silently in the direction people don't check. A prompt architecture built on Grok 4.3's 1M window — whole-repo dumps, full session history replayed each turn, long document sets in one pass — does not fit 4.5.

On any Grok prompt review, ask which version is actually in use before recommending long-context patterns. If the artifact assumes 1M and the target is 4.5, that's a `[CRITICAL]` finding: the fix is chunking, retrieval or summarization of the carried context, not rewording.

### The 200K pricing step makes prompt bloat directly expensive

Grok is now the vendor where a fat persistent-context file has a visible price cliff rather than a gradual cost. A system prompt plus `AGENTS.md` plus accumulated history that crosses 200K flips the whole request to double rate — including the output tokens.

Practical review consequence: on Grok 4.5, "trim the persistent context" stops being hygiene advice and becomes a budget argument. The atlas's standing target (under 8 KiB of load-bearing rules) is nowhere near the cliff on its own; the risk is in what the harness *accumulates* around it.

### Wording-side behaviors

- Family rules apply unchanged — outcome-first, no reasoning-depth prose (now that `reasoning_effort` exists: set the parameter out-of-band, don't write "think harder"), tool guidance in tool descriptions
- xAI published **no prompting guide** for 4.5; there is no documented wording delta versus 4.3. Don't invent one — the differences that matter for a review are the window, the price step, and the reasoning-effort knob

## Grok 4.3 (April 30 / May 4, 2026 — previous flagship, 1M context)

### Headline facts

- **1M token context** (jumped from earlier Grok 4.x)
- **Native video input**
- **40% price cut** vs Grok 4.20
- **#1 on agentic tool calling** (Artificial Analysis leaderboard at release)
- **Intelligence Index 53** (median 35), #1 on CaseLaw v2 and CorpFin
- **8 legacy Grok models retire May 15, 2026** — check pinned model strings in production prompts
- **Grok Skills feature** for persistent expertise

### Wording-side behaviors

- Apply family rules above
- Outcome-first / action-biased framing aligns with the model
- Don't write reasoning-depth prose — model decides
- Use the 1M context deliberately (full task history, prior outputs, code state)
- Tool descriptions carry most of the per-tool guidance

### When Grok 4.3 specific tuning helps

- Cost-sensitive agentic loops where the 40% input price cut shifts the economics
- Workloads where leaderboard-leading tool-call quality matters more than reasoning depth
- Video-input tasks (other current frontier vendors trail or don't support)
- **Workloads that genuinely need a 1M window on Grok** — 4.3 is now the only Grok that has one

### When NOT to invest in Grok-specific tuning

- Abstract reasoning / HLE-style problems — Claude / GPT / Gemini Pro tier still ahead
- Tasks heavily dependent on broad factual recall (Grok has its own bias profile — not extensively benchmarked publicly)
- Cross-vendor prompts where Grok is tertiary — family rules align well with GPT/Gemini, so a cross-vendor compromise prompt usually runs cleanly on Grok without dedicated tuning
