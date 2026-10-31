# Model-specific wording differences — xAI Grok family

What changes about how you should PHRASE prompts for **Grok 4.7** (xAI's current recommended model, September 2026) and **Grok 4.6** (previous). This file is about wording — not API parameters, context sizes or pricing, except where one of those changes prompt design (the 500K window, the 200K price step). Grok 4.5 and 4.3 are still callable (no shutdown date published) but their sections live in `archive/models/grok.md`.

xAI publishes **no text-model prompting guide** — its docs index lists prompting guides only for Image and Speech-to-Speech. The behaviors below come from xAI's model, reasoning, moderation and cookbook pages, plus release notes and independent reviews. **Grok 5 has not shipped** (as of 2026-10-31 the newest model post on x.ai/news is Grok 4.7) — don't tune prompts for it.

---

## Family-wide rules (Grok 4.x)

### 1. Built-in reasoning — `reasoning_effort` is the depth lever; from 4.5 on it cannot be disabled

On **grok-4.5, grok-4.6 and grok-4.7** reasoning is always on: *"If not specified, `reasoning_effort` defaults to `"high"`. Reasoning cannot be disabled."* Ladder: `low` / `medium` / `high` on 4.5; `low` / `medium` / `high` / `xhigh` on 4.6 and 4.7 (`xhigh` sent to 4.5 is treated as `high`). **Grok 4.3 is the exception:** its model page lists `none`, `low`, `medium`, `high`, `xhigh`, default `low` — so "reasoning cannot be disabled" is a 4.5+ rule, not a family-wide one. On `grok-4.20-multi-agent` the same parameter sets the number of agents, not depth.

**Wording implication:**
- Don't write "think step by step" — the model already reasons internally
- If reasoning is shallow on a task, the lever is `reasoning_effort` **out-of-band**, not a prose line. xAI's own eval is a caution against reflexively raising it: *"More reasoning doesn't always pay off"* — on its sample every effort scored 87–93% and the best one changed from run to run. Measure before recommending `xhigh`
- "Don't think" / "answer without reasoning" is **structurally unimplementable** on Grok 4.5 / 4.6 / 4.7 — same class as the thinking-forced-on cases in `antipatterns.md` #38. On 4.3 it is still inert prose: the off-switch there is `reasoning_effort: "none"`, not a sentence
- Harness note: *"`presencePenalty`, `frequencyPenalty`, and `stop` cannot be used with reasoning models. Requests that include them return an error."* A prompt port that relied on stop sequences to cut output needs a format instruction instead
- Verify-against-criteria phrasing works; "think harder" doesn't — the cross-vendor rule (`techniques.md` §30)

### 2. Action-biased — outcome-first works well

Grok 4.x is biased toward **doing**, not describing (Grok 4.3 led the Artificial Analysis agentic tool-calling leaderboard at its release; 4.6 and 4.7 are framed by xAI around long-running agents). Prompts that prescribe step-by-step actions tend to be followed faithfully; prompts that describe what *should happen* let the model find a working path.

Closer to the GPT-5.x outcome-first direction than to Claude's appreciate-step-naming direction. When in doubt, lean outcome-first.

### 3. Grok Skills — persistent custom expertise

xAI's "Skills" feature (introduced with Grok 4.3) gives persistent custom expertise across conversations — roughly Claude Code's Skill system: pre-loaded capability shims invoked by triggers.

**Wording implication:** if the prompt under review is a Grok Skill description, treat trigger phrasing the same way as Claude SKILL.md descriptions — explicit when-to-invoke clauses, scope boundary, bilingual keywords if the domain is non-English.

### 4. Multimodal input

xAI's model pages list text + image → text for the API models (4.3 and 4.7 alike). Video input, slide generation and document/spreadsheet editing were reported as product features at the 4.3 launch (secondary sources), not as API modalities.

**Wording implication:** a prompt written for a text-only model should be audited for image-handling cases ("you cannot see images" caveats are wrong here).

### 5. Long-context patterns apply — but check the window per version

Standard long-context patterns:
- Documents at top, question at bottom
- Quote-grounded answers
- Permission to say "I don't know"
- Stable content first for prompt caching

⚠️ **The window is not monotonic in this family.** Grok 4.3 has 1M; **Grok 4.5, 4.6 and 4.7 have 500K** (xAI's models page). Moving from 4.3 to any later Grok halves the window — see § Grok 4.6 › The migration trap.

### 6. OpenAI-compatible API surface

xAI's API mirrors OpenAI's chat-completions shape (`messages`, `tools`, `response_format`) and also serves the Responses API. Most prompts written for OpenAI's SDK semantics port directly to Grok without restructuring.

### 7. Persona / "act as" — neutral, no strong directional preference documented

No clear evidence from xAI docs or independent reviews that persona blocks help or hurt on Grok 4.x. Functional persona ("You are a Python expert") is safe; identity-credential personas ("You are a Stanford-trained statistician") are likely neutral noise.

When porting from a Gemini prompt (where persona gives +5%), don't expect the same lift. When porting from a GPT-5.x prompt (where persona hurts), don't expect the same penalty either.

### 8. Tone — terse, direct

Grok's marketed personality is more conversational than other frontier models, but for agent-facing prompts the **default behavior is terse and direct**, similar to GPT-5.x. If you need warmth or conversational pacing in end-user-facing copy, ask explicitly.

### 9. Tool guidance in tool descriptions — same cross-vendor rule

Put tool-specific guidance inside tool descriptions, not the system prompt. Same as the GPT-5.x / Gemini / DeepSeek recommendation. Costs nothing if untrue; load-bearing if true.

### 10. Content moderation has a request parameter — `guidelines` (October 2026)

xAI's Custom Moderation Controls: *"Add a `guidelines` array to a request to set how Grok further handles specific content categories."* Categories are `nsfw`, `violence`, `profanity`; *"Today the only behavior is `decline`. Categories you leave out continue to keep the model's default moderation behavior."* *"Grok follows the guidelines as it writes its response, in its own words."*

**Wording implication:** system-prompt prose like "never use profanity", "no graphic violence", "keep it free of sexual content" has a structured home on Grok. When the reviewer can reach the harness, raise moving those rules into `guidelines` as a conversation point (not an `[ADD]` to the prompt). Keep the prose when:
- the prompt is cross-vendor — no other vendor in the atlas has this parameter;
- the rule is narrower or different from the three categories (brand-specific taboo topics, tone rules) — the parameter covers only those three, with `decline` only;
- the harness can't guarantee the array on every call: *"Guidelines apply only to the request that carries them. To apply them to a whole conversation, send them on every request in it."*

Not available on multi-agent models or deferred completions; a request with no server-side tools and no files can't combine `guidelines` with `stop` or `n > 1` (400). xAI does not say whether the parameter outranks conflicting prompt prose — don't leave the two contradicting each other.

### 11. xAI cookbook measurements that touch prompt layout

First-party cookbook pages (dated 2026-10-05) carry two measured findings worth applying in a review:
- **Volatile text at the top of the system prompt breaks caching.** *"Putting the current time at the top of the system prompt made the same chat cost 2.2 to 2.9 times as much in our runs."* The fix in the same page: put the time at the top of the latest message. The same applies to any per-turn value in a "stable" prefix. Also: *"A compaction item has to come first, because the API drops anything before it, `instructions` included"* — a system prompt placed before a compaction item is lost.
- **Prompt-injection fencing is necessary but not sufficient.** The tested defense tells the model that tool output is data and wraps fetched pages in `<untrusted>` tags. The vendor's caveat: *"The fence wraps pages in tags that a page can close itself."* Each defense alone missed at least one of ten attacks; only the four together (fence, split steps, tool gate, id check) stopped all ten. A prompt that relies on a fence line alone is under-defended.

---

## Grok 4.7 (September 2026 — xAI's recommended model)

`grok-4.7`, released September 21, 2026. xAI's models page: *"For everything else, including code, use Grok 4.7. It is the most capable model we've built."* (still the recommendation as of 2026-10-31). The release post ([x.ai/news/grok-4-7](https://x.ai/news/grok-4-7)) frames it as *"our most capable model for coding and knowledge work"* that *"works longer on difficult tasks, checks its own work more carefully"*, and says it was *"trained to natively understand the Grok Bot harness, making it better at conversational tasks and general knowledge work."* No prompting guide — the delta versus 4.6 is behavioral emphasis and the parameter surface, not documented prose changes.

### Headline facts

- **500K context** — same as 4.5 / 4.6, so the 1M → 500K migration trap (§ Grok 4.6) still applies to a 4.3-era architecture
- **`reasoning_effort`: `low` / `medium` / `high` (default) / `xhigh`** — reasoning cannot be disabled
- **Encrypted reasoning is always returned** on the Responses API, and xAI now says what to do with it: *"pass them back unchanged in the next request's `input` so the model keeps its reasoning across turns even when you manage conversation history yourself."* Harness consequence: a loop that rebuilds history should replay the reasoning items rather than strip them; Chat Completions has no field for it
- Image input is listed on the models page. A "Fast" variant was reported for Cursor / Grok Build only, not the public API — secondary source

### Wording-side consequences

- Family rules apply unchanged (outcome-first, no reasoning-depth prose, tool guidance in tool descriptions). "Works longer and checks its own work" is vendor framing: treat a carried-over "double-check before finishing" line as *untested* on 4.7 rather than as stripped — no first-party source says the model over-verifies the way Opus 5 does.
- The 4.6 review points on the window, the price step and the migration trap carry over as written.

---

## Grok 4.6 (August 2026 — previous recommended model)

Released August 12, 2026; xAI's recommended model until Grok 4.7 (September 21). Focus on long-running agents, interactive and visual work.

### Headline facts

- **500K context** — down from Grok 4.3's 1M; 4.6 did not bring back 1M
- **Knowledge cutoff:** February 1, 2026
- **Two-step pricing by prompt length:** a request whose prompt reaches 200K tokens is billed at roughly double the rate for *all* its tokens (same on 4.5 and 4.7) — kept here because it changes prompt design (below)
- **`reasoning_effort`: `low` / `medium` / `high` (default) / `xhigh`** — reasoning cannot be disabled
- **Long-running agents are the stated focus** — self-testing and verification on longer trajectories, stronger first passes on visual/interactive projects

### The migration trap: newer model, smaller window

This is the rare case where an upgrade **narrows** a capability, and it fails silently in the direction people don't check. A prompt architecture built on Grok 4.3's 1M window — whole-repo dumps, full session history replayed each turn, long document sets in one pass — does not fit 4.6 (or 4.5 / 4.7).

On any Grok prompt review, ask which version is actually in use before recommending long-context patterns. If the artifact assumes 1M and the target is 4.5 / 4.6 / 4.7, that's a `[CRITICAL]` finding: the fix is chunking, retrieval or summarization of the carried context, not rewording.

### The 200K pricing step makes prompt bloat directly expensive

Grok is the vendor where a fat persistent-context file has a visible price cliff rather than a gradual cost. A system prompt plus `AGENTS.md` plus accumulated history that crosses 200K flips the whole request to the higher rate — including the output tokens.

Practical review consequence: on Grok 4.7 / 4.6 / 4.5, "trim the persistent context" stops being hygiene advice and becomes a budget argument. The atlas's standing target (under 8 KiB of load-bearing rules) is nowhere near the cliff on its own; the risk is in what the harness *accumulates* around it.

### Wording-side behaviors

- Family rules apply — outcome-first, no reasoning-depth prose (the lever is `reasoning_effort`), tool guidance in tool descriptions
- No prompting guide for 4.6; the delta versus 4.5 is the parameter surface (`xhigh`), not prose. Don't invent one — the differences that matter for a review are the window, the price step, the reasoning-effort knob and the vendor recommendation

---

## Cross-vendor rules (when Grok is one of several targets)

Grok 4.7 fits cleanly into a cross-vendor compromise prompt because most of its documented behaviors align with the **GPT-5.x + Gemini 3.x intersection**:

| Axis | Direction on Grok | Cross-vendor alignment |
|---|---|---|
| Persona | neutral | matches GPT (hurts) less than Gemini (+5%) — safe to drop or keep functional |
| Step prescription | tolerated; outcome-first preferred | matches GPT-5.x outcome-first direction |
| Reasoning lever | `reasoning_effort` (`low`/`medium`/`high`/`xhigh` on 4.6 / 4.7, `low`/`medium`/`high` on 4.5, default `high`) — **cannot be disabled on 4.5+**; 4.3 accepts `none` (default `low`) | knob is out-of-band, not prose — aligns with the other vendors' parameters (the one prose-level control in the atlas is Muse Spark Open's `Reasoning strength: N.` line, `meta.md`) |
| Tool guidance location | tool description | matches all current frontier vendors |
| Output format | OpenAI-compatible API surface | safe with `json_schema` across vendors |
| Content moderation | `guidelines` request parameter (`nsfw` / `violence` / `profanity` → `decline`) | Grok-only — keep moderation rules in prose in a cross-vendor prompt |
| Long context | **500K on 4.5 / 4.6 / 4.7, 1M on 4.3** | Grok 4.5+ is the *binding* window in a cross-vendor set that includes it — size the carried context for 500K |
| Aggressive emphasis | likely inert | matches GPT / Gemini / Kimi / GLM / Qwen / DeepSeek inert behavior |

If a cross-vendor prompt already works on Claude + GPT + Gemini, expect it to also work on Grok 4.7 with minimal adjustment.

---

## Source notes

xAI publishes less detailed prompting guidance than Anthropic / OpenAI / Google. The behaviors documented here come from:

- xAI models page ([docs.x.ai/developers/models](https://docs.x.ai/developers/models), read 2026-08-18, 2026-09-24, re-read 2026-10-31 raw) — Grok 4.7 recommendation sentence, 500K on 4.5 / 4.6 / 4.7, 1M on 4.3, the 200K two-step pricing rule, Feb 1 2026 cutoff (4.6); no shutdown dates for 4.3 / 4.5 / 4.6
- Grok 4.3 model page ([docs.x.ai/developers/models/grok-4.3](https://docs.x.ai/developers/models/grok-4.3.md), read 2026-10-31 raw) — reasoning efforts `none`/`low`/`medium`/`high`/`xhigh`, default `low`; text + image → text
- xAI reasoning docs ([docs.x.ai/developers/model-capabilities/text/reasoning](https://docs.x.ai/developers/model-capabilities/text/reasoning.md), read 2026-09-24, re-read 2026-10-31 raw) — `reasoning_effort` for 4.5 / 4.6 / 4.7, default `high`, "Reasoning cannot be disabled", `xhigh` on 4.6+; `presencePenalty` / `frequencyPenalty` / `stop` error on reasoning models; 4.7 encrypted reasoning always returned and "pass them back unchanged"; multi-agent effort = agent count
- xAI Custom Moderation Controls ([docs.x.ai/developers/advanced-api-usage/custom-moderation-controls](https://docs.x.ai/developers/advanced-api-usage/custom-moderation-controls.md), October 2026 release note, read 2026-10-31 raw) — `guidelines` array, categories, `decline`, per-request scope, unsupported request types, errors
- xAI cookbook, read 2026-10-31 raw: [prompt caching](https://docs.x.ai/cookbook/examples/prompt-caching.md) (time at top of system prompt 2.2–2.9× cost; compaction item first), [prompt-injection defenses](https://docs.x.ai/cookbook/examples/prompt-injection-defenses.md) (`<untrusted>` fence and its caveat; four defenses together), [reasoning-effort evals](https://docs.x.ai/cookbook/examples/reasoning-effort-evals.md) (87–93% across efforts)
- xAI docs index ([docs.x.ai/llms.txt](https://docs.x.ai/llms.txt), read 2026-10-31 raw) — prompting guides exist only for Image and Speech-to-Speech
- x.ai/news (rendered, read 2026-10-31 raw) — newest model post is "Introducing Grok 4.7"; no Grok 5 or 4.8. A secondary claim that 4.8 / 4.9 come before Grok 5 is search-snippet only and not used here
- xAI's Grok 4.7 release post ([x.ai/news/grok-4-7](https://x.ai/news/grok-4-7), read 2026-09-24) and Grok 4.6 release post ([x.ai/news/grok-4-6](https://x.ai/news/grok-4-6)) — framing quotes and launch dates
- Artificial Analysis post on the Grok 4.3 release ([artificialanalysis.ai/articles/xai-launches-grok-4-3](https://artificialanalysis.ai/articles/xai-launches-grok-4-3-with-improved-agentic-performance-and-lower-pricing)) — agentic tool-calling leaderboard lead; product features at the 4.3 launch from independent reviews (TimesOfAI, ChatlyAI, Releasebot)

Several axes are deliberately marked neutral or `?` — they haven't been documented as strongly as on other vendors. When confidence matters, validate against your task.
