# Model-specific wording differences — xAI Grok family

What changes about how you should PHRASE prompts for **Grok 4.6** (xAI's current recommended model, August 2026), **Grok 4.5**, **Grok 4.3**, and the lineage leading to them. Companion to other model files in this directory.

Coverage here is deliberately compact — xAI publishes less explicit prompting guidance than Anthropic / OpenAI / Google, and many of the documented behaviors are inferred from release notes and independent reviews rather than first-party docs.

---

## Family-wide rules (Grok 4.x)

### 1. Built-in reasoning — reasoning cannot be disabled, `reasoning_effort` is the depth lever

Grok 4.x ships with built-in reasoning that **cannot be disabled**. The model decides reasoning depth autonomously based on the task.

**Since Grok 4.5 the depth lever exists:** xAI documents `reasoning_effort` for **grok-4.5 and grok-4.6** (default `high`; `low` / `medium` / `high` on 4.5, plus `xhigh` on 4.6). This is the same class of runtime knob as Claude's `effort` / OpenAI's `reasoning_effort` / Gemini's `thinking_level` / DeepSeek's `thinking` — no longer an absent parameter (older atlas text saying "no reasoning-effort parameter documented" is stale).

**Wording implication:**
- Don't write "think step by step" — model already does this internally
- If reasoning quality is shallow on a specific task, the lever is `reasoning_effort` **out-of-band** (API parameter), not a prose line
- "Don't think" / "answer without reasoning" is **structurally unimplementable** on Grok 4.5 / 4.6 — same class of antipattern as the Kimi/GLM thinking-forced-on cases (`antipatterns.md` #38)
- Same family rule as Gemini 3.x and Claude — verify-against-criteria phrasing works; "think harder" doesn't

### 2. Action-biased — outcome-first works well

Grok 4.3 leads the Artificial Analysis agentic tool-calling leaderboard. The model is biased toward **doing**, not describing. Prompts that prescribe step-by-step actions tend to be followed faithfully; prompts that describe what *should happen* let the model find a working path.

Closer to GPT-5.5 outcome-first direction than to Claude's appreciate-step-naming direction. When in doubt, lean outcome-first.

### 3. Grok Skills — persistent custom expertise

Grok 4.3 introduced a "Skills" feature for persistent custom expertise across conversations. Acts roughly like Claude Code's Skill system — pre-loaded capability shims invoked by triggers.

**Wording implication:** if the prompt review is a Grok Skill description, treat trigger phrasing the same way as Claude SKILL.md descriptions — explicit when-to-invoke clauses, scope boundary, bilingual keywords if domain is non-English.

### 4. Native video, slide generation, agentic tools

Grok 4.3 added native video input, slide generation, document/spreadsheet editing tools. Means tool descriptions may travel through additional modality channels.

**Wording implication:** when a prompt assumes text-only context, audit for video/image-handling cases that Grok 4.3 can take but a Grok 4.2-era prompt didn't anticipate.

### 5. Long-context patterns apply — but check the window per version

Standard long-context patterns:
- Documents at top, question at bottom
- Quote-grounded answers
- Permission to say "I don't know"
- Stable content first for prompt caching

⚠️ **The window is not monotonic in this family.** Grok 4.3 has 1M; **Grok 4.5 and Grok 4.6 have 500K** (both figures from xAI's own models page). Moving to a newer Grok *halves* the window — see the Grok 4.6 and 4.5 sections.

### 6. OpenAI-compatible API surface

xAI's API mirrors OpenAI's chat-completions shape (`messages`, `tools`, `response_format`). Means most prompts written for OpenAI's SDK semantics port directly to Grok without restructuring.

### 7. Persona / "act as" — neutral, no strong directional preference documented

No clear evidence from xAI docs or independent reviews that persona blocks help or hurt on Grok 4.x specifically. Functional persona ("You are a Python expert") is safe; identity-credential personas ("You are a Stanford-trained statistician") are likely neutral noise.

When porting from a Gemini prompt (where persona gives +5%), don't expect the same lift. When porting from a GPT-5.5 prompt (where persona hurts), don't expect the same penalty either.

### 8. Tone — terse, direct

Grok's marketed personality is more conversational than other frontier models, but for agent-facing prompts the **default behavior is terse and direct**, similar to GPT-5.5. If you need warmth or conversational pacing in end-user-facing copy, ask explicitly.

### 9. Tool guidance in tool descriptions — same cross-vendor rule

Put tool-specific guidance inside tool descriptions, not the system prompt. Same as GPT-5.5 / Gemini / DeepSeek recommendation. Costs nothing if untrue; load-bearing if true.

---

## Grok 4.6 (August 2026 — xAI's recommended model)

xAI's models page names it the default choice: *"For everything else, including code, use Grok 4.6. It is the most intelligent and fastest model we've built."* Released August 12, 2026, with a focus on long-running agents, interactive and visual work. Matches **GPT-5.6 Sol on the Artificial Analysis Intelligence Index (61)**, and tops Grok 4.5 on DeepSWE v1.1 (65.9 vs 54), FrontierCode (61.3 vs 56.6), Terminal-Bench v3.0 (26 vs 15.7) among others (vendor-reported vs competitors; third-party figures from published system cards / leaderboards).

### Headline facts

- **500K context** — down from Grok 4.3's 1M; the 4.3 → 4.5 migration trap did NOT get fixed by a 1M return
- **Knowledge cutoff:** February 1, 2026
- **Two-step pricing by prompt length:** $2.00 / $6.00 per M tokens (in / out) below 200K tokens of prompt, **doubling at 200K and above**; **cached input $0.50 / M tokens** (xAI docs)
- **`reasoning_effort`: `low` / `medium` / `high` (default) / `xhigh`** — controls reasoning depth; **reasoning cannot be disabled** (xAI reasoning docs). "High" reasoning is the default
- **Long-running agents are the stated focus** — self-testing and verification on longer trajectories, stronger first passes on visual/interactive projects

### The migration trap: newer model, smaller window

This is the rare case where an upgrade **narrows** a capability, and it fails silently in the direction people don't check. A prompt architecture built on Grok 4.3's 1M window — whole-repo dumps, full session history replayed each turn, long document sets in one pass — does not fit 4.6 (or 4.5).

On any Grok prompt review, ask which version is actually in use before recommending long-context patterns. If the artifact assumes 1M and the target is 4.5 / 4.6, that's a `[CRITICAL]` finding: the fix is chunking, retrieval or summarization of the carried context, not rewording.

### The 200K pricing step makes prompt bloat directly expensive

Grok is the vendor where a fat persistent-context file has a visible price cliff rather than a gradual cost. A system prompt plus `AGENTS.md` plus accumulated history that crosses 200K flips the whole request to double rate — including the output tokens.

Practical review consequence: on Grok 4.6 / 4.5, "trim the persistent context" stops being hygiene advice and becomes a budget argument. The atlas's standing target (under 8 KiB of load-bearing rules) is nowhere near the cliff on its own; the risk is in what the harness *accumulates* around it.

### Wording-side behaviors

- Family rules apply — outcome-first, no reasoning-depth prose (the lever is the `reasoning_effort` parameter), tool guidance in tool descriptions
- xAI published **no dedicated prompting guide** for 4.6; the wording delta versus 4.5 is the parameter surface, not prose. Don't invent one — the differences that matter for a review are the window, the price step, the reasoning-effort knob, and the vendor recommendation

---

## Grok 4.5 (July 2026 — previous recommended model)

xAI's models page previously named it the default choice; it's now superseded by Grok 4.6 (August 2026). Secondary sources put its release at July 8, 2026.

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

---

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

---

## Grok 5 (not yet released — Q2 2026 target as of May 2026)

xAI confirmed Grok 5 as upcoming with reported 6 T total parameters and MoE architecture. Q1 2026 launch missed; Q2 2026 window is the current guidance. **Don't tune prompts for Grok 5 yet** — public capability details are speculative as of this writing.

When Grok 5 ships, prompt-atlas should re-evaluate this file. Confirmed features expected from xAI's roadmap (dynamic agent spawning, persistent memory, multimodal unification) will likely add new wording considerations.

---

## Cross-vendor rules (when Grok is one of several targets)

Grok 4.3 fits cleanly into a cross-vendor compromise prompt because most of its documented behaviors align with the **GPT-5.5 + Gemini 3.x intersection**:

| Axis | Direction on Grok | Cross-vendor alignment |
|---|---|---|
| Persona | neutral | matches GPT (hurts) less than Gemini (+5%) — safe to drop or keep functional |
| Step prescription | tolerated; outcome-first preferred | matches GPT-5.5 outcome-first direction |
| Reasoning lever | built-in, cannot be disabled; `reasoning_effort` (`low`/`medium`/`high`/`xhigh` on 4.6, `low`/`medium`/`high` on 4.5, default `high`) | knob is out-of-band, not prose — aligns with every other vendor's parameter |
| Tool guidance location | tool description | matches all current frontier vendors |
| Output format | OpenAI-compatible API surface | safe with `json_schema` across vendors |
| Long context | **500K on 4.5 / 4.6, 1M on 4.3** | Grok 4.5 / 4.6 is the *binding* window in a cross-vendor set that includes it — size the carried context for 500K |
| Aggressive emphasis | likely inert | matches GPT / Gemini / Kimi / GLM / Qwen / DeepSeek inert behavior |

If a cross-vendor prompt already works on Claude + GPT + Gemini, expect it to also work on Grok 4.3 with minimal adjustment.

---

## Source notes

xAI publishes less detailed prompting guidance than Anthropic / OpenAI / Google. The behaviors documented here come from:

- xAI's own models page ([docs.x.ai/developers/models](https://docs.x.ai/developers/models), read 2026-08-18) — Grok 4.6 recommendation ("For everything else, including code, use Grok 4.6"), 500K context, Feb 1 2026 cutoff, the two-step pricing threshold at 200K, and Grok 4.3's 1M window. The July 8, 2026 release date for 4.5 is from secondary coverage; the 4.6 launch date (August 12, 2026) is from xAI's release post
- xAI's reasoning docs ([docs.x.ai/developers/model-capabilities/text/reasoning](https://docs.x.ai/developers/model-capabilities/text/reasoning)) — **`reasoning_effort` documented for `grok-4.5` and `grok-4.6`** (`low`/`medium`/`high`, default `high`; `xhigh` on 4.6), reasoning cannot be disabled. This corrects earlier atlas text that claimed no reasoning-effort parameter existed for 4.5
- xAI's Grok 4.6 release post ([x.ai/news/grok-4-6](https://x.ai/news/grok-4-6)) — focus on long-running agents, AA Intelligence Index 61 (parity with GPT-5.6 Sol Max), $2/$6 pricing, DeepSWE v1.1 65.9, Terminal-Bench v3.0 26
- xAI model pricing page ([docs.x.ai/developers/pricing](https://docs.x.ai/developers/pricing)) — cached-input rate $0.50/M
- Artificial Analysis post on Grok 4.3 release ([artificialanalysis.ai/articles/xai-launches-grok-4-3](https://artificialanalysis.ai/articles/xai-launches-grok-4-3-with-improved-agentic-performance-and-lower-pricing))
- Apiyi.com migration guide for Grok 4.3 API
- Independent reviews (TimesOfAI, ChatlyAI, Releasebot)
- xAI's release notes referenced via Releasebot

Several axes are deliberately marked neutral or `?` — they haven't been documented as strongly as on other vendors. When confidence matters, validate against your task.
