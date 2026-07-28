# Model-specific wording differences — xAI Grok family

What changes about how you should PHRASE prompts for **Grok 4.5** (xAI's current recommended model, July 2026), **Grok 4.3**, and the lineage leading to them. Companion to other model files in this directory.

Coverage here is deliberately compact — xAI publishes less explicit prompting guidance than Anthropic / OpenAI / Google, and many of the documented behaviors are inferred from release notes and independent reviews rather than first-party docs.

---

## Family-wide rules (Grok 4.x)

### 1. Built-in reasoning — no separate toggle documented

Grok 4.3 ships with built-in reasoning. The model decides reasoning depth autonomously based on the task. There's no equivalent of Claude's `effort` / OpenAI's `reasoning_effort` / Gemini's `thinking_level` / DeepSeek's three-level `thinking` parameter that the user can dial.

**Wording implication:**
- Don't write "think step by step" — model already does this internally
- If reasoning quality is shallow on a specific task, the lever is **task framing** (more specific success criteria, clearer constraints) rather than a depth parameter
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

⚠️ **The window is not monotonic in this family.** Grok 4.3 has 1M; **Grok 4.5 has 500K** (both figures from xAI's own models page). Upgrading to the newer model *halves* the window — see the Grok 4.5 section.

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

## Grok 4.5 (July 2026 — xAI's recommended model)

xAI's models page names it the default choice: *"For everything else, including code, use Grok 4.5. It is the most intelligent and fastest model we've built."* Release date isn't stated on the page; secondary sources put it at July 8, 2026.

### Headline facts

- **500K context** — down from Grok 4.3's 1M
- **Knowledge cutoff:** February 1, 2026
- **Two-step pricing by prompt length:** $2.00 / $6.00 per M tokens (in / out) below 200K tokens of prompt, **doubling to $4.00 / $12.00 at 200K and above**
- **No reasoning-effort parameter documented** — family rule #1 still holds: the model decides depth

### The migration trap: newer model, smaller window

This is the rare case where an upgrade **narrows** a capability, and it fails silently in the direction people don't check. A prompt architecture built on Grok 4.3's 1M window — whole-repo dumps, full session history replayed each turn, long document sets in one pass — does not fit 4.5.

On any Grok prompt review, ask which version is actually in use before recommending long-context patterns. If the artifact assumes 1M and the target is 4.5, that's a `[CRITICAL]` finding: the fix is chunking, retrieval or summarization of the carried context, not rewording.

### The 200K pricing step makes prompt bloat directly expensive

Grok is now the vendor where a fat persistent-context file has a visible price cliff rather than a gradual cost. A system prompt plus `AGENTS.md` plus accumulated history that crosses 200K flips the whole request to double rate — including the output tokens.

Practical review consequence: on Grok 4.5, "trim the persistent context" stops being hygiene advice and becomes a budget argument. The atlas's standing target (under 8 KiB of load-bearing rules) is nowhere near the cliff on its own; the risk is in what the harness *accumulates* around it.

### Wording-side behaviors

- Family rules apply unchanged — outcome-first, no reasoning-depth prose, tool guidance in tool descriptions
- xAI published **no prompting guide** for 4.5; there is no documented wording delta versus 4.3. Don't invent one — the differences that matter for a review are the window, the price step, and the vendor recommendation

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
| Reasoning lever | built-in, no toggle | doesn't conflict with any vendor's parameter |
| Tool guidance location | tool description | matches all current frontier vendors |
| Output format | OpenAI-compatible API surface | safe with `json_schema` across vendors |
| Long context | **500K on 4.5, 1M on 4.3** | Grok 4.5 is now the *binding* window in a cross-vendor set that includes it — size the carried context for 500K |
| Aggressive emphasis | likely inert | matches GPT / Gemini / Kimi / GLM / Qwen / DeepSeek inert behavior |

If a cross-vendor prompt already works on Claude + GPT + Gemini, expect it to also work on Grok 4.3 with minimal adjustment.

---

## Source notes

xAI publishes less detailed prompting guidance than Anthropic / OpenAI / Google. The behaviors documented here come from:

- xAI's own models page ([docs.x.ai/docs/models](https://docs.x.ai/docs/models), read 2026-07-28) — Grok 4.5 recommendation, 500K context, Feb 1 2026 cutoff, the two-step pricing threshold at 200K, and Grok 4.3's 1M window. **No prompting guide and no reasoning-effort parameter are documented for 4.5.** The July 8, 2026 release date is from secondary coverage only
- Artificial Analysis post on Grok 4.3 release ([artificialanalysis.ai/articles/xai-launches-grok-4-3](https://artificialanalysis.ai/articles/xai-launches-grok-4-3-with-improved-agentic-performance-and-lower-pricing))
- Apiyi.com migration guide for Grok 4.3 API
- Independent reviews (TimesOfAI, ChatlyAI, Releasebot)
- xAI's release notes referenced via Releasebot

Several axes are deliberately marked neutral or `?` — they haven't been documented as strongly as on other vendors. When confidence matters, validate against your task.
