# Model-specific wording — Meta Muse family

What changes about how you should PHRASE prompts for **Muse Spark 1.2** (Meta Superintelligence Labs, August 5, 2026 — updated from 1.1, July 9), **Muse Code** (August 5, coding specialist) and **Muse Glimmer** (August 10, open-weights 30B local agentic). Companion to the other model files in this directory.

Coverage here is deliberately short. Meta has published **no prompting guide** for the family: the facts below come from the launch posts and the Meta Model API / dev docs, and everything not stated there stays `?`. That's the honest position, and it's why this file is a page rather than a chapter.

**What changed since the July 2026 entry.** The atlas previously carried Muse Spark 1.1 as "closed weights, public-preview API, most axes `?`". Version **1.2** (05.08) keeps the same public-preview surface; **Muse Code** (05.08) is a coding-specialist sibling known mostly by availability; **Muse Glimmer** (10.08) opens a new lane — an Apache-2.0 30B model sized for one consumer GPU, trained for local agentic work. The behavioral axes (persona, few-shot, literalism, emphasis) are still undocumented across the family — don't fill them in by analogy with Llama, which these models are not successors to.

---

## Family-wide rules

### 1. It manages its own context window — don't hand-roll what it already does

Meta's launch post (1.1; behavior carried into 1.2): *"Muse Spark can actively manage its context window of 1 million tokens."* The model pulls earlier material back in and compacts as it goes, rather than relying on the harness to trim history.

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

## Muse Spark 1.2 (August 5, 2026 — current API flagship)

### Headline facts

- **1M context**, actively self-managed (behavior carried from 1.1)
- **Multimodal** — images, video, PDF
- **Meta Model API** (public preview); OpenAI-compatible; structured output; parallel tool calls
- **Thinking mode** available in the Meta AI app and on meta.ai
- Closed weights — an API vendor surface, not an open-weights story

### What changed from 1.1

The 05.08 update (with Muse Code) moved the family forward, but Meta published **no prompting-relevant delta** for 1.2 itself — the family rules above, written against 1.1, apply unchanged. Treat 1.2 as the current nameplate for the same behavioral picture: context self-management, goal conditioning, delegation support, `?`-behavioral axes.

## Muse Code (August 5, 2026 — coding specialist)

Facts only: a coding-specialist sibling released alongside Spark 1.2. No prompting guide, no documented behavioral delta vs Spark 1.2 — record so a later pass doesn't re-investigate, and don't invent version/role-specific wording (same treatment as Mistral Medium 3.5's facts-only coverage). If a review targets "Muse Code" explicitly, apply the family rules and mark unspecified axes `?`.

## Muse Glimmer (August 10, 2026 — open-weights 30B local agentic model)

Meta's first **open-weight agentic model** for local deployment: `Muse-Glimmer-30B` on Hugging Face, **Apache 2.0**, sized to run on a Mac/PC with a single consumer GPU (~17-19GB quantized, 4-bit). 30B total parameters — above the atlas's Class 2 range (2-9B), but local-first; treat it as a Class-1-adjacent open model with its own documented capabilities.

### Headline facts

- **30B dense**, distilled from Muse Spark outputs, trained for agentic long-horizon work; RL + distillation across general/reasoning/coding/agentic domains
- **Tool use first-class**: function calling with precise schemas across extended workflows; **failure recovery** (diagnoses a failed call and retries rather than halting)
- **Multimodal input** (perception encoder — screenshots, charts, documents alongside text)
- **Controllable effort** — "different reasoning strengths" documented: a *parameter/serving* lever, not prose (`reasoning-depth` rule applies: never write "think harder" in the body)
- **Scaffold compatibility**: OpenClaw and other orchestration patterns; integrations via llama.cpp, MLX, ExecuTorch, Ollama, LM Studio, Unsloth; vLLM / SGLang at scale; speculative-decoding drafter for speed
- **Multilingual** (>100 languages), 512K-class context training (per the training schedule) — long-context claims to be validated per-surface

### Wording behaviors that matter

- **Family rules #1 and #2 carry over** — don't hand-roll context management it already does; name the goal rather than the steps (goal conditioning)
- **Agentic prompts should name tools and retry expectations plainly** — failure recovery is a documented capability, so "if a tool errors, diagnose and retry" scaffolding the model already has is a candidate for removal (same class as `antipatterns.md` #36)
- **Controllable effort is a serving-time knob** — surface it as a parameter question (quantized runtimes differ), never as `[ADD]` prose
- **Behavioral axes stay `?`** — no prompting guide; persona / few-shot / emphasis / literalism cells are guesses, so cross-vendor prompts should let other vendors decide those cells

### When Muse Glimmer specific tuning helps

- Local / offline agent workflows (screenshots, tool loops, file work) where cloud round-trips are unwanted
- LLM-as-judge and local coding agents at the 30B-with-one-GPU tier

### What is still `?`

Persona tolerance, few-shot behavior, literalism, aggressive-emphasis response, step-by-step prescription, subagent-spawn default, verbosity default — same set as Spark; marked `?` in the matrix and treated conservatively in reviews (`principles.md` § universal-prompt rules).

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

Because so many cells are `?`, adding Muse to a cross-vendor prompt costs little in *compromise* but buys little in *optimization*. It doesn't introduce a new opposite-default axis — it introduces unknowns.

---

## Source notes

- Meta's launch post, [Introducing Muse Spark 1.1 and the Meta Model API](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/) (read 2026-07-28) — 1M self-managed context, multimodality, the planning-mode / goal-conditioning / subagent-delegation / context-compaction list, public-preview API, Thinking mode.
- **Muse Spark 1.2 / Muse Code** (2026-08-05) — dated per Meta research index / secondary coverage; Meta published no prompting-relevant release note for the wording layer, which is why 1.2 inherits the 1.1 chapter unchanged. Fact-status: availability confirmed, behavioral delta not documented.
- [Introducing Muse Glimmer: An Open Agentic Model That Runs on Your Device](https://research.meta.ai/blog/introducing-muse-glimmer-open-agentic-model) (2026-08-10, read 2026-08-29) — 30B, Apache 2.0, one-GPU sizing (~17GB quantized), tool-calling + failure recovery, multimodal perception encoder, controllable effort, OpenClaw compatibility, speculative decoding (DFlash drafter), llama.cpp/MLX/ExecuTorch integrations.
- The "clean OpenAI-compatible package", structured output and parallel tool calling wording in that post comes from an **early-partner testimonial** (Replit's CEO), not from Meta's own specification text. Treated here as an availability claim, not as a behavioral guarantee.
- **No prompting guide exists.** Every behavioral axis not listed above is `?` on purpose. If a review needs one of them, the answer is "test it", not "assume Llama" — these models are not Llama successors and there is no migration path from Llama 4.
