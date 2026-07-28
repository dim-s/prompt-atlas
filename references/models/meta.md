# Model-specific wording — Meta Muse Spark

What changes about how you should PHRASE prompts for **Muse Spark 1.1** (Meta Superintelligence Labs, July 9, 2026) — Meta's first model that is addressable like any other API vendor rather than a closed preview. Companion to the other model files in this directory.

Coverage here is deliberately short. Meta has published **no prompting guide** for Muse Spark: the facts below come from the launch post for the model and the Meta Model API, and everything not stated there stays `?`. That's the honest position, and it's why this file is a page rather than a chapter.

**What changed since the April 2026 entry.** The atlas previously carried Muse Spark as "closed-weight, limited docs, most axes `?`", with no dedicated file. Version 1.1 opened a public-preview API, so the model became reviewable: prompts targeting it now exist in the wild, and several capability axes are documented. The behavioral axes (persona, few-shot, literalism, emphasis) are still undocumented — don't fill them in by analogy with Llama, which Muse Spark is not a successor to.

---

## Family-wide rules

### 1. It manages its own context window — don't hand-roll what it already does

Meta's launch post: *"Muse Spark 1.1 can actively manage its context window of 1 million tokens."* The model pulls earlier material back in and compacts as it goes, rather than relying on the harness to trim history.

**Wording implication:** the scaffolding many long-run prompts carry — "summarize the conversation so far every N turns", "keep a running state block and re-read it", "drop older context when you approach the limit" — is a candidate for removal here. Same class of finding as stripping "after every 3 tool calls, summarize progress" from a Claude prompt: paying prompt budget for behavior the model already has (`antipatterns.md` #36).

Not a licence to bloat the input. Self-management is about the *conversation*, not about a persistent-context file that is re-read on every turn.

### 2. Agentic features are documented as supported — so name them plainly

The post states the model *"performs well with popular agentic coding setups, supporting common features like planning mode, goal conditioning, subagent delegation, and context compaction."*

**Wording implication:** these are the four levers a Muse Spark agent prompt can lean on by name. In particular, *goal conditioning* — stating the objective the run is optimizing for, rather than the steps — is the vendor's own framing, so an outcome-first prompt is the safe default. Delegation is supported, but the *default* propensity (does it spawn readily, or does it undertrigger like Opus 4.8?) is **undocumented** — state what you want explicitly rather than assuming either direction.

### 3. OpenAI-compatible surface — prompts port structurally, not behaviorally

The Meta Model API is presented as an OpenAI-compatible package with structured output and parallel tool calling. A prompt written against the OpenAI SDK shape runs here without restructuring: same roles, same tools array, same schema-driven output contract.

That's a *format* guarantee, not a behavior guarantee. Don't infer GPT-5.x wording defaults (strip persona, zero-shot, outcome-first-or-else) from API compatibility — those are OpenAI model behaviors, not properties of the message format.

### 4. Multimodal by design

Perception, multimodal reasoning and tool use are the post's headline claims, with visual-to-code generation and long-form image/video captioning called out. Prompts that carry "you cannot see images" caveats from a text-only model are wrong here.

### 5. Reasoning depth — surface, don't embed

A "Thinking" mode exists in the Meta AI app and on meta.ai. Whether the API exposes an equivalent depth parameter is **not documented in the launch post** — `?`. Until it is: don't write reasoning-depth prose into the body (the universal rule), and treat depth as a question for the user's API config rather than something the prompt can set.

---

## Muse Spark 1.1 (July 9, 2026)

### Headline facts

- **1M context**, actively self-managed
- **Multimodal** — images, video, PDF
- **Meta Model API in public preview**; OpenAI-compatible; structured output; parallel tool calls
- **Thinking mode** available in the Meta AI app and on meta.ai
- Closed weights — this is an API vendor now, not an open-weights story

### What is still `?`

Persona tolerance, few-shot behavior, literalism, aggressive-emphasis response, step-by-step prescription, subagent-spawn default, verbosity default, sampling-parameter policy. Marked `?` in the matrix and to be treated conservatively in reviews (`principles.md` § universal-prompt rules): write for the strictest neighbor in the set rather than guessing Meta's direction.

### When Muse Spark specific tuning helps

Rarely, today. If the user is targeting it, the useful moves are: strip context-management scaffolding (rule #1), use goal-conditioned outcome-first framing (rule #2), and push the output contract into structured output. Everything beyond that is unfounded until Meta publishes prompting guidance.

---

## Cross-vendor rules (when Muse Spark is one of several targets)

| Axis | Direction on Muse Spark | Cross-vendor alignment |
|---|---|---|
| Context management scaffolding | model self-manages — strip | costs nothing elsewhere; other vendors tolerate its absence |
| Goal / outcome framing | vendor's own framing (goal conditioning) | matches GPT-5.x and Gemini; safe on Claude |
| Tool guidance location | `?` — no guidance published | use the cross-vendor default: inside tool descriptions |
| Output format | structured output supported | safe with `json_schema` across vendors |
| Persona, few-shot, emphasis, literalism | `?` | let the other vendors in the set decide these cells |
| Reasoning depth | `?` at the API level | universal rule holds: never in the body |

Because so many cells are `?`, adding Muse Spark to a cross-vendor prompt costs little in *compromise* but buys little in *optimization*. It doesn't introduce a new opposite-default axis — it introduces unknowns.

---

## Source notes

- Meta's launch post, [Introducing Muse Spark 1.1 and the Meta Model API](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/) (read 2026-07-28) — 1M self-managed context, multimodality, the planning-mode / goal-conditioning / subagent-delegation / context-compaction list, public-preview API, Thinking mode.
- The "clean OpenAI-compatible package", structured output and parallel tool calling wording in that post comes from an **early-partner testimonial** (Replit's CEO), not from Meta's own specification text. Treated here as an availability claim, not as a behavioral guarantee.
- **No prompting guide exists.** Every behavioral axis not listed above is `?` on purpose. If a review needs one of them, the answer is "test it", not "assume Llama" — Muse Spark is not a Llama successor and there is no migration path from Llama 4.
