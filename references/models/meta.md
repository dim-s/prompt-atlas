# Model-specific wording — Meta Muse family

What changes about how you should PHRASE prompts for Meta's Muse models: hosted **Muse Wonder 1.0** (October 12, 2026 — Meta's most capable model, preview), hosted **Muse Spark 1.3** (September 2, 2026 — current Spark, "Recommended for new work") and **1.2** (August 5, previous), and the open-weight **Muse Spark Open** (October 15, 2026, 816B-A42B) and **Muse Glimmer** (August 10, 30B). This file is about wording — not API parameters, context sizes or pricing, except where one of those changes prompt design. Companion to the other model files in this directory.

**Which models have a prompting guide.** Meta has published official prompting guides for the two **open-weight** models — **Muse Spark Open** and **Muse Glimmer** (dev.meta.ai). The hosted models (**Wonder, Spark 1.3 / 1.2**) still have none: their facts come from launch posts and the Model API docs (models, reasoning, tool calling), and behavioral axes not stated there stay `?`. Don't carry a guide's advice from Spark Open to hosted Spark or the other way: Meta calls Spark Open *"the open-weight release of Muse Spark"* but gives it no version number and a different reasoning surface.

**Muse Code is not a model.** It is Meta's coding agent for the terminal and CI — *"a coding agent built for Muse Spark"* — running the hosted models (default `muse-spark-1.2`; Wonder and Wonder Full selectable). Older atlas text listed it as a "coding-specialist sibling" model; that was wrong. A review that targets "Muse Code" targets whichever model it runs.

The behavioral axes (persona, few-shot, literalism, emphasis) are undocumented for the hosted models — don't fill them in by analogy with Llama, which these models are not successors to.

---

## Family-wide rules

### 1. It manages its own context window — don't hand-roll what it already does

Meta's launch post (Spark 1.1; carried forward): *"Muse Spark can actively manage its context window of 1 million tokens."* The model pulls earlier material back in and compacts as it goes, rather than relying on the harness to trim history.

**Wording implication:** the scaffolding many long-run prompts carry — "summarize the conversation so far every N turns", "keep a running state block and re-read it", "drop older context when you approach the limit" — is a candidate for removal on hosted Spark / Wonder. Same class of finding as stripping "after every 3 tool calls, summarize progress" from a Claude prompt (`antipatterns.md` #36). Not a licence to bloat the input: self-management is about the *conversation*, not a persistent-context file re-read every turn. **Not documented for Glimmer** — its guide says the opposite for long loops: *"prune or summarize old turns before you hit the configured limit."*

### 2. Agentic features are documented as supported — so name them plainly

The 1.1 post: the model *"performs well with popular agentic coding setups, supporting common features like planning mode, goal conditioning, subagent delegation, and context compaction."*

**Wording implication:** *goal conditioning* — stating the objective the run optimizes for rather than the steps — is the vendor's own framing, so outcome-first is the safe default. Delegation is supported, but the *default* propensity (spawns readily or undertriggers) is **undocumented** — state what you want.

### 3. OpenAI-compatible surface — prompts port structurally, not behaviorally; forced tool use is not available

The Meta Model API serves Responses, Chat Completions and Messages protocols. A prompt written against the OpenAI SDK shape runs here without restructuring — a *format* guarantee, not a behavior one. Don't infer GPT-5.x wording defaults from API compatibility.

Two tool-calling facts change prompt wording on the hosted models (tool-calling docs):
- *"`tool_choice` must be `"auto"`"* — `"none"`, `"required"` and named function choices return HTTP 400. **The prompt has to say when a tool applies**; there is no API way to force a call (same fix as Fable 5.1 / Opus 5.5 / Sonnet 5.5 in `claude.md`).
- `strict` *"controls only schema validation"*; *"the model's generated arguments are guided by your `parameters` schema but are not guaranteed to validate against it"* — validate in code; don't promise the user schema-perfect calls in the prompt.

### 4. Multimodal by design

Wonder and Spark take text, image, video, audio and PDF; Spark Open text, image, video, audio; Glimmer text and image. Prompts that carry "you cannot see images" caveats from a text-only model are wrong here. One exception on the hosted side: *"Audio understanding in Muse Spark 1.3 is not fully supported"* — Meta points audio work to Spark 1.2.

### 5. Reasoning depth — a parameter on every Muse model, never prose (one documented exception)

| Model | Knob | Ladder | Off? |
|---|---|---|---|
| Wonder 1.0, Spark 1.3 / 1.2 (hosted) | `reasoning_effort` on Chat Completions; `reasoning.effort` on the Responses API | `minimal` · `low` · `medium` · `high` · `xhigh` · `max` (`max`: Wonder and `muse-spark-1.3`, not the contributor variant); omitted → *"a model-determined level"* | **no** — `"none"` → HTTP 400 |
| Spark Open (self-hosted) | `reasoning_effort` (request or `chat_template_kwargs`) | `low` · `medium` · `high` · `xhigh` (default `xhigh`) | **no** — *"Reasoning can't be turned off"* |
| Glimmer (self-hosted) | `reasoning_strength` template argument | `xhigh` · `high` · `medium` · `low` (default `high`) | no off value documented |

The two hosted spellings are **one knob placed by protocol**, not two names to reconcile: *"On Chat Completions, use the top-level `reasoning_effort` parameter. On the Responses API, nest it as `reasoning.effort`."* The open models differ in ladder and, for Glimmer, in the argument name.

**Wording implication:** "don't think" / "answer immediately" is unimplementable on hosted Muse and on Spark Open — strip it (`antipatterns.md` #38 class). Depth complaints are the parameter. The vendor's own advice for the hosted models: *"For direct-answer tasks, such as lookups, formatting, and translation, set `reasoning_effort` to `"low"`."* **The exception:** on Spark Open a `Reasoning strength: N.` line in the system prompt is a documented control that overrides the parameter — see § Muse Spark Open. It is the only model in the atlas where an effort-like line in prose is a real setting.

**Harness note (hosted):** on Chat Completions `reasoning_content` *"is redacted to empty before the response reaches an external caller, so there is nothing to replay and each turn reasons from scratch."* Multi-turn agents that need reasoning carried over belong on the Responses API (`previous_response_id` or encrypted reasoning).

---

## Muse Wonder 1.0 (October 12, 2026 — Meta's most capable model, preview)

`muse-wonder-1.0`, Model API (Standard mode only) and Muse Code. Meta: *"Meta's most capable model, built for complex software engineering and long-horizon agentic work"*, *"following detailed instructions closely"*. 1,048,576-token context. *"Muse Wonder is in preview."* A delta on Spark 1.3 — *"Muse Wonder and Muse Spark take the same requests on the same endpoints, so moving a workload between them is a model ID change."*

### What differs from Spark 1.3

- **Wonder Full has no API id.** *"Muse Wonder Full, the higher-capability variant, is available only in Muse Code with a subscription, so setting reasoning effort to `"max"` on Muse Wonder doesn't select it."* A prompt or harness that assumes `max` means "the Full model" is wrong.
- **Same collaboration behavior as Spark 1.3.** The launch post on "Active Focus": *"Along the way, it asks for clarification when needed and checks in before taking important actions."* The § Spark 1.3 consequences for non-interactive runs apply unchanged — state that no one can answer mid-task, what is pre-authorized, and the ambiguity fallback.
- **"Active Memory"** — *"carrying important context about you across conversations"* — is a product-side claim; no API control or prompt guidance is published for it (`?`).
- Family rules #3 (no forced `tool_choice`) and #5 (ladder `minimal`…`max`, no `none`) apply. Meta's docs use Wonder for long-running agent and sub-agent orchestration examples; for those, use the Responses API so reasoning carries across turns.

### What is still `?`

No prompting guide. Persona tolerance, few-shot behavior, literalism, aggressive emphasis, step prescription, subagent-spawn default, any control over the collaboration behavior — same set as Spark 1.3.

---

## Muse Spark 1.3 (September 2, 2026 — current hosted Spark)

### Headline facts

- `muse-spark-1.3` and `muse-spark-1.3-contributor` on the Meta Model API (the contributor tier's prompts are used to improve Meta's products — a data-use question, not a wording one; it also lacks the `max` effort level). Meta's models page: *"Recommended for new work."*
- 1M context and native multimodal perception — family rules #1 and #4 hold (audio: use 1.2)
- No prompting guide — everything below is quoted from the launch post or the Model API docs, or derived from them

### The behavioral shift: it asks, and it confirms

Meta's post: *"Trained to more actively collaborate with the user, Muse Spark 1.3 asks clarifying questions when prompts are ambiguous, invokes help from the user when stuck, and confirms before taking consequential actions."* And: *"When working on long tasks, it adapts to user preferences, either providing frequent updates or working silently in the background."* The safety section adds that *"On complex agentic tasks, the model has better calibration on what constitutes irreversible actions and proceeds accordingly."*

Meta documents the behavior, not how to steer it. The wording consequences below are **derived**, and they cut two ways:

- **Interactive agents** — the behavior is the feature. Don't bolt on generic "ask me before anything risky" scaffolding; name only the project-specific actions that need confirmation, since the model's own notion of "consequential" isn't published.
- **Autonomous / non-interactive runs** (batch jobs, headless execution, no one to answer) — a clarifying question or a confirmation pause stalls the run. State it plainly: that no one is available to answer mid-task, which reversible actions are pre-authorized, and what to do on ambiguity (state the assumption and continue, or stop with a report). Whether a system-prompt instruction reliably suppresses the collaboration behavior is **undocumented — test it**; no parameter for it is published.
- **Update cadence** — because it adapts to what it reads as the user's preference, say which you want (frequent progress notes or silent background work) instead of leaving it to inference.
- **Ambiguity now triggers a question rather than a guess.** Where a best guess is acceptable, say so.

Same direction as GPT-6 Astra, released the same week: OpenAI documents the same stall on autonomous runs and ships snippets for it (`gpt.md § GPT-6 Astra`). Those snippets are OpenAI-model guidance — usable as a starting shape here, not validated on Muse.

### Other documented deltas from 1.2

- **Long instructions hold better** — *"Across multi-step tasks, it’s better at preserving detailed requirements without dropping constraints or drifting from the requested workflow."* Constraint restatements added to keep 1.2 on track are trim candidates — re-test before removing.
- **Multitasking in one thread** — *"it now more accurately maps incoming prompts to the correct task within messy, single-threaded contexts"*.
- **Limits awareness** — trained to recognize *"when it hits hurdles instead of hallucinating outcomes"*.
- **Coding** — *"it takes fewer turns where not needed and is less verbose"*; ~20% fewer tool calls and ~25% fewer tokens in Meta engineers' comparisons. Concision lines ported from 1.2 coding prompts may now overcorrect (same class as GPT-5.6 / Gemini 3.6 Flash).
- **Adversarial robustness** — *"stronger adversarial robustness, with improved resistance to adversarial inputs and prompt injections"*.

### Reasoning depth

Now documented — family rule #5 (hosted row). Never in the body.

### What is still `?`

Persona tolerance, few-shot behavior, literalism, aggressive-emphasis response, step-by-step prescription, subagent-spawn default, any control surface for the collaboration behavior.

## Muse Spark 1.2 (August 5, 2026 — previous hosted Spark)

`muse-spark-1.2` (and `-contributor`) — Meta's models page: *"the previous version"*; still the default model in Muse Code and Meta's pick for audio input. 1M context, self-managed; multimodal; same Model API surface and reasoning ladder as 1.3 except `max`. Meta published **no prompting-relevant delta** for 1.2 itself — the family rules, written against 1.1, apply unchanged: context self-management, goal conditioning, delegation support, `?` behavioral axes.

---

## Muse Spark Open (October 15, 2026 — open weights, 816B-A42B)

`meta-models/Muse-Spark-Open-816B-A42B` (BF16 and NVFP4; Hugging Face repo gated with manual approval). 816B total / 42B active, 1M context, text + image + video + audio → text, >100 languages, "Muse Model License with limited commercial terms". Not on the Model API — self-host (SGLang, vLLM, TensorRT-LLM, llama.cpp) or a hosted provider. Meta: *"the open-weight release of Muse Spark"*; no Spark version number is given, so the Spark 1.3 collaboration delta is **not** documented for it. The card adds: *"This open weights version has additional safety post-training to make it more robust against misuse as a standalone model without system guardrails."*

Unlike the hosted models, it has an **official prompting guide** and a recommended-settings page. The guide's wording rules, verbatim:

### Rules first — the vendor names the attention pattern

*"Muse Spark Open pays more attention to the beginning of the system prompt, so put the rules that must always hold first."* The guide's skeleton: `You are [specific role].` → `## Objective` → `## Priority rules` (numbered invariants, highest severity first, then a tie-breaker) → `## Scope check` → `## Decision procedure` (ending in *"Do not infer [unsupported info]."*) → `## Tools` (*"Use native tool calls. Use [tool] when [condition]. Do not invent results."*) → `## Output` (*"Return exactly: {…}"*) → `## Completion` (*"Stop when all fields supported. If evidence unavailable, return [explicit unknown]."*).

**Review consequence:** on Spark Open, an invariant buried mid-prompt or in a closing "important reminders" block is a placement finding, not a style note — move it to the top. Background, examples and long context go after the rules.

### Don't restate the tools; the template writes its own system lines

- *"You don't need to describe your tools in the system prompt. Pass them in `tools`, and the template adds their definitions."* Prose that re-describes the `tools` array is a strip candidate; keep only the *when* (`Use [tool] when [condition]`).
- *"Your system prompt doesn't replace the template's own system block; the template adds it to that block."* The rendered order is: the default lines (`You are a helpful AI assistant.`, knowledge cutoff 2026-01-04, current date) → your system prompt → tool definitions → reasoning strength (*"unless your system prompt sets one"*) → the list of recipients. Derived consequences: your prompt is never the first line the model reads, so open with a specific role that overrides the generic one (the guide's skeleton does); a hand-written "today is …" line duplicates the template's date (set it with `current_date` in `chat_template_kwargs` instead).

### `Reasoning strength: N.` — the one prose line that is a real control

*"To fix the strength for every request instead, set it in the system prompt with the number, not the level name, for example `Reasoning strength: 256.` A `Reasoning strength` line in the system prompt takes precedence over `reasoning_effort`."* The map is `low` → 8, `medium` → 32, `high` → 256, `xhigh` (default) → 512.

This is a documented exception to the atlas-wide rule that effort lines in prose are inert (`SKILL.md` § Reasoning-depth knobs). Treat it as config that happens to live in the prompt:
- **Legitimate** when the operator deliberately pins depth for every request — then it must be a number.
- **A trap in ported prompts** (derived from the chat template, which is public): the template rewrites "Reasoning effort" → "Reasoning strength" in your system prompt, and if the result contains `reasoning strength:` anywhere it skips its own numeric line. So a line such as `Reasoning effort: high` carried over from another model's prompt becomes `Reasoning strength: high` — a level name, which the guide says not to use — and silently overrides the request's `reasoning_effort`. On a Spark Open review, any "reasoning effort / strength:" text in the system prompt is a high-severity finding: delete it, or replace it with the intended number.

The default is `xhigh`; the settings page: *"set `reasoning_effort` explicitly when you want shorter reasoning."* Card: *"It is recommended to use Reasoning Effort of high or xhigh for complex problem solving, coding, and agentic tasks."*

### Tools, media, output

- Native tool calls are XML (ATEM): `<atem:function_calls><atem:invoke name="…"><atem:parameter name="…">…`; results come back in `<tool_output name="…">`. Servers started with the `muse` parsers convert both to standard `tool_calls` — a prompt shouldn't describe a JSON call format of its own. Parallel calls are supported; *"A result carries only the function name, so when several calls use the same tool, send the results in the order of the calls"* (harness).
- Media order is meaningful: *"The model sees the parts in the order you send them, so put each piece of media where it belongs relative to the text that refers to it."* For video: *"use clips longer than a few seconds at 448 px or more on the short side."*
- Harness: *"Never stop on <|eom|>"* — it ends a message mid-turn and truncates tool calls; the card asks developers to *"avoid exposing raw chain-of-thought"*.

### What is still `?`

Few-shot behavior, aggressive-emphasis response, literalism, subagent-spawn default — the guide doesn't address them.

---

## Muse Glimmer (August 10, 2026 — open-weights 30B local agentic model)

Meta's first **open-weight agentic model** for local deployment: `Muse-Glimmer-30B` on Hugging Face, **Apache 2.0**, sized to run on a Mac/PC with a single consumer GPU (~17-19GB quantized, 4-bit). 30B dense — above the atlas's Class 2 range, so treat it as a Class-1-adjacent open model. Distilled from Muse Spark; tool use with failure recovery; image input; >100 languages; default context 128K (*"The model supports longer contexts"*).

It has an **official prompting guide** (dev.meta.ai/docs/muse-glimmer/prompting).

### Wording rules from the guide (verbatim)

- *"Be specific about the task. "You are a Python code reviewer" outperforms "You are a helpful assistant.""* — a specific functional role is the vendor's recommendation; a generic helper line is a weak opener.
- *"State constraints upfront: output format, length limits, tone."*
- *"Avoid meta-instructions about how the model works internally."* — lines like "you are a reasoning model, think in your hidden channel first" are strip candidates.
- *"Muse Glimmer supports one tool call per turn. It does not support parallel tool calls"* — strip "call all the tools you need in parallel / batch your reads" lines ported from Claude or Spark prompts.
- Reasoning is built into the format (a private `to=self` turn); depth is the `reasoning_strength` template argument (family rule #5), never prose. Note the different name from Spark Open's `reasoning_effort`.

### Other carried facts

- **Family rule #2 carries over** — name the goal rather than the steps (goal conditioning). Family rule #1 does **not** — prune or summarize old turns in long loops (guide).
- **Failure recovery is a documented capability**, so "if a tool errors, diagnose and retry" scaffolding is a candidate for removal (same class as `antipatterns.md` #36).
- Template matters more than wording at this size: *"manually formatting the prompt without `apply_chat_template` leads to degraded output quality."*

### When Muse Glimmer specific tuning helps

- Local / offline agent workflows (screenshots, tool loops, file work) where cloud round-trips are unwanted
- LLM-as-judge and local coding agents at the 30B-with-one-GPU tier

### What is still `?`

Few-shot behavior, literalism, aggressive-emphasis response, step-by-step prescription, subagent-spawn default, verbosity default — the guide doesn't address them; treated conservatively in reviews (`principles.md`).

---

## Cross-vendor rules (when a Muse model is one of several targets)

| Axis | Direction on Muse | Cross-vendor alignment |
|---|---|---|
| Context management scaffolding | hosted Spark / Wonder self-manage — strip; Glimmer — keep pruning | costs nothing elsewhere; other vendors tolerate its absence |
| Goal / outcome framing | vendor's own framing (goal conditioning) | matches GPT-5.x and Gemini; safe on Claude |
| Rule placement | Spark Open: rules that must always hold **first** (vendor) | safe everywhere — put invariants at the top of a shared prompt |
| Persona | Glimmer / Spark Open guides: a specific role beats a generic helper; hosted: `?` | matches the cross-vendor functional-persona default |
| Tool guidance location | Spark Open: tools in `tools`, prose says only *when*; hosted: say when a tool applies (no forced `tool_choice`) | use the cross-vendor default: inside tool descriptions, plus a when-to-use line |
| Parallel tool calls | supported on hosted and Spark Open; **not on Glimmer** | don't require batching in a prompt that may run on Glimmer |
| Output format | structured output supported; `strict` doesn't guarantee valid arguments | safe with `json_schema`; validate in code |
| Reasoning depth | parameter on every Muse model; no off-switch on hosted Muse or Spark Open (none documented for Glimmer); Spark Open's `Reasoning strength: N.` line is the one prose control | never "don't think" in a shared prompt; keep any `Reasoning strength` line out of prompts that also target other models |
| Few-shot, emphasis, literalism | `?` | let the other vendors in the set decide these cells |

---

## Source notes

- Meta's launch post, [Introducing Muse Spark 1.1 and the Meta Model API](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/) (read 2026-07-28) — 1M self-managed context, multimodality, the planning-mode / goal-conditioning / subagent-delegation / context-compaction list, public-preview API.
- **Muse Spark 1.2** (2026-08-05) — dated per Meta research index / secondary coverage; no prompting-relevant release note. Models page (below) calls it "the previous version".
- [Introducing Muse Glimmer](https://research.meta.ai/blog/introducing-muse-glimmer-open-agentic-model) (2026-08-10, read 2026-08-29) — 30B, Apache 2.0, one-GPU sizing, tool-calling + failure recovery, multimodal perception encoder, controllable effort, speculative decoding, runtime integrations.
- The "clean OpenAI-compatible package" wording in the Glimmer post is an **early-partner testimonial** (Replit's CEO), not Meta's specification text.
- [Introducing Muse Spark 1.3](https://research.meta.ai/blog/introducing-muse-spark-1-3) (2026-09-02, read 2026-09-13) — the collaboration behavior, long-form instruction reliability, single-thread multitasking, limits awareness, coding efficiency vs 1.2, irreversible-action calibration and prompt-injection robustness.
- [Muse Spark — Meta Model API model page](https://developer.meta.com/ai/models/muse-spark/) (read 2026-09-13) — IDs `muse-spark-1.3` / `muse-spark-1.3-contributor`, contributor-tier data use.
- Meta Model API docs, read 2026-10-31 (raw): [models](https://dev.meta.ai/docs/models.md) (six model lines; Wonder description, preview status, 1,048,576 context, Wonder Full availability; Spark 1.3 "Recommended for new work", 1.2 "the previous version", 1.3 audio caveat; Spark Open "the open-weight release of Muse Spark", no Model API endpoint; "same requests on the same endpoints"), [reasoning](https://dev.meta.ai/docs/reasoning.md) (ladder `none`→400 … `max`; Chat Completions vs Responses placement; omitted = model-determined; direct-answer tasks → `low`; Chat Completions reasoning redacted), [tool calling](https://dev.meta.ai/docs/tool-calling.md) (`tool_choice` must be `"auto"`; `strict` semantics; parallel calls default `true`), [overview](https://dev.meta.ai/docs/overview.md) (Muse Code "a coding agent built for Muse Spark"), [Muse Code configuration](https://dev.meta.ai/docs/muse-code/configuration.md) (default model `muse-spark-1.2`; `--reasoning-effort` incl. client-side `ultra`; Meta provider rejects `none`), [docs index](https://dev.meta.ai/llms.txt) (prompting guides exist for Spark Open and Glimmer only).
- [Muse Wonder](https://research.meta.ai/blog/muse-wonder) launch post (2026-10-12, read 2026-10-31 raw) — Wonder and Wonder Full, Active Memory, Active Focus ("asks for clarification when needed and checks in before taking important actions").
- Muse Spark Open, read 2026-10-31 (raw): [announcement](https://research.meta.ai/blog/muse-spark-open) (2026-10-15; 816B / 42B active, 1M, licence, hosting partners), [Hugging Face card](https://huggingface.co/meta-models/Muse-Spark-Open-816B-A42B) (public HTML page incl. the embedded chat template; recommended parameters and effort, safety post-training, chain-of-thought exposure note), [prompting guide](https://dev.meta.ai/docs/muse-spark-open/prompting.md) (system block order, rules first, skeleton, tools not described in prose, effort map, `Reasoning strength` line precedence, ATEM calls, media order), [recommended settings](https://dev.meta.ai/docs/muse-spark-open/settings.md) (sampling, effort default `xhigh`, never stop on `<|eom|>`). The ported-prompt trap is derived from the chat template's string replacement and its `reasoning strength:` check, not stated by Meta.
- [Muse Glimmer prompting guide](https://dev.meta.ai/docs/muse-glimmer/prompting.md) (read 2026-10-31 raw; publication date not established) — specific role, constraints upfront, no meta-instructions, `reasoning_strength`, one tool call per turn, 128K default context, prune old turns, template pitfalls.
- Every hosted-model behavioral axis not listed above is `?` on purpose. If a review needs one, the answer is "test it", not "assume Llama" — these models are not Llama successors and there is no migration path from Llama 4.
