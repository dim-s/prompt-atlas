# Model-specific wording differences — Mistral frontier family

What changes about how you should PHRASE prompts for **Mistral Large 4** (October 2026, current flagship; open weights since 29.10) and the rest of the Mistral frontier line: **Mistral Small 4** (March 2026), the **Ministral 3** edge series, and two models that are still callable but retiring — **Mistral Large 3** and **Mistral Medium 3.5** (both deprecated 2026-10-31, retirement 2027-04-30, replacement Large 4). This file is about wording — not API parameters, context sizes or pricing, except where one of those changes prompt design.

**This file covers Mistral frontier only.** Smaller Ministral 3B / 8B / 14B edge variants and older Ministral / Mistral 7B-class variants live in `small-local.md`. Don't confuse them — the prompting regime differs (frontier tolerates abstract principles; small Ministral regresses on the same wording).

---

## Family-wide rules

### 1. Functional persona helps — close to Claude / Kimi direction

Mistral docs and community guides converge on **functional persona being helpful**, similar to Claude's direction (neutral-to-helpful) and Kimi's (officially endorsed). Not as load-bearing as Gemini's +5% identity boost, but no penalty like on GPT-5.x.

When porting from a Claude prompt, persona lines usually travel cleanly to Mistral frontier models without changes.

### 2. Standard OpenAI-compatible chat format

Mistral uses the standard `system` / `user` / `assistant` / `tool` role structure. Tool calling is OpenAI-compatible (`tools` array, function definitions). No surprise format requirements — except the chunked reasoning output in rule #3.

### 3. Reasoning is a per-request knob on the frontier models; only Ministral 3 splits it into variants

Mistral's reasoning docs list `reasoning_effort` on **Small 4** (`mistral-small-2603`), **Medium 3.5** (`mistral-medium-3-5`) and **Large 4**, with different ladders:
- **Small 4 / Medium 3.5:** `high` (full thinking chunk before the answer) and `none` (*"The model thinks minimally and the thinking chunk is omitted from the response"*); for Medium 3.5, *"For agentic and code use cases, `reasoning_effort="high"` is recommended."*
- **Large 4:** `low` / `medium` / `high`, default `high`; `none` is not accepted, so reasoning cannot be switched off (§ Mistral Large 4).

The **Ministral 3** edge series is the part of the family that still ships separate Instruct / Reasoning / Base variants per size — there the lever is model choice. The older Magistral native-reasoning models are deprecated.

With reasoning on, `message.content` comes back as a **list of chunks** (a `thinking` chunk, then a `text` chunk with the answer), not a plain string. The docs: *"always replay the full assistant message (including `ThinkChunk`) back into the message history. Dropping the reasoning trace across turns degrades model performance."*

**Wording implication:**
- Depth complaints are a `reasoning_effort` setting (or, on Ministral 3, a variant choice), not prose
- Don't write "think step by step" — same family rule as other current frontier vendors
- "Answer without thinking" is unimplementable on Large 4 (lowest step is `low`); on Small 4 / Medium 3.5 the off-switch is `none`, not a sentence
- A harness that keeps only the text of past assistant turns is dropping something the vendor says matters — flag it (harness, not wording)

### 4. Few-shot helps — no GPT-5.x-style penalty

Mistral tolerates and benefits from few-shot examples for format steering. 3–5 examples is a reasonable default, similar to Claude. Don't strip few-shot blocks when porting from Claude prompts.

### 5. Context — 1M on Large 4, 256K on the retiring models

Long-context patterns apply: documents top, question bottom, "Based on the above..." anchor. Large 4 has a 1M window; Large 3 and Medium 3.5 have 256K. A prompt architecture moving from Large 3 to Large 4 gains room, so the window stops being the reason for chunking — but the 1M size alone is no evidence of quality at 1M (untested in the atlas).

### 6. Open weights — licences differ per model

Mistral is the European open-weights frontier story: Large 4 under the **Mistral Ecosystem Open-Weight License** (*"a modified version of the MIT license … operating a Model-as-a-Service business requires a separate commercial license with Mistral"*), Medium 3.5 under a modified MIT licence, Ministral 3 under Apache 2.0. Production deployments may run self-hosted, on Mistral's API, or through partner clouds.

**Wording-side this doesn't change anything** — same model, multiple hosts. But it matters for prompt portability: prompts that work on the API surface should also work on self-hosted vLLM/SGLang/llama.cpp deployments without changes.

### 7. Tone — direct, similar to GPT-5.x / Mistral's historical character

Default tone is task-oriented and terse. Mistral doesn't have Claude's validation-forward warmth or Gemini's structured-but-concise. Ask explicitly for warmth or extended explanation if needed.

### 8. Image understanding

Large 4 (text + image → text), Large 3 and the Ministral 3 variants take images. When porting older Mistral 7B-era prompts that assumed text-only, audit for new vision-handling cases.

---

## Mistral Large 4 (October 2026 — current flagship)

`mistral-large-4` / `mistral-large-4-0`. Preview from 2026-10-06; on 2026-10-29 Mistral released the improved model and its weights — `mistralai/Mistral-Large-4-1T-A52B` (plus `-NVFP4`), ungated on Hugging Face. ~1T total parameters, *"49 billion active parameters per token (52 billion including embeddings and output layers)"*, 1M context, text + image input. The docs model card still says **"Status: Public Preview"** (read 2026-10-31), even though the blog describes the model as available "today". Replacement for both Large 3 and Medium 3.5.

### The reasoning ladder — what changed from the preview

- **`reasoning_effort`: `low` / `medium` / `high`, default `high`.** The docs: *"Mistral Large 4 does not accept `none`, `minimal`, `xhigh`, or `max`; these are not aliases for its supported levels."* The open-weights chat template raises an error on any other value.
- **Preview users:** *"Use `low` for the level previously called `none`."* So the preview's no-reasoning mode is gone — **reasoning cannot be switched off**. The vendor's own guidance per level (HF card): *"Use `reasoning_effort="high"` for complex prompts and agentic usage, `'low'` for fast chat replies and `'medium'` for day to day usage."*
- With reasoning, the response is the chunk list from family rule #3, and the full assistant message must be replayed. Whether `low` returns a plain string or a chunk list is not stated (`?`) — parse both. The chat template rejects a message that carries a thinking chunk *and* a top-level `reasoning` field at once (harness).

### Wording guidance — the only official lines

No separate prompting guide. The model card's Recommended Settings carry the vendor's wording advice, verbatim:

- *"**System Prompt**: define a clear environment and use case, including guidance on how to effectively leverage tools in agentic systems."*
- *"**Tools**: keep the set of tools well-defined and limit their number to the minimum required for the use case."*

and a capability claim: *"**System Prompt**: strong adherence and support for system prompts."*

**Review consequences:**
- A Large 4 system prompt should say what environment the model is in and what it is for; in an agent, how to use the tools — not just a role line. A prompt that is only a persona plus rules is missing what the vendor asks for first.
- An oversized tool list is a vendor-named problem here: trim tools the use case doesn't need before adding prose about when to use them.
- Strong system-prompt adherence means family rule #1 (persona) and rule #4 (few-shot) carry over; there is no evidence that aggressive emphasis is needed — keep it for real invariants.

### Migrating from Large 3 or Medium 3.5

- Depth is now `reasoning_effort` on the same model; re-check prose that compensated for the instruct model's lack of reasoning ("work through this carefully step by step") — the lever is the parameter.
- Remove any "don't think / answer immediately" line — unimplementable on Large 4. If latency is the complaint, it's `low`.
- Window grows 256K → 1M; the carried context no longer has to be cut for size.
- Harness: content becomes chunked when reasoning is on; replay the full assistant message.

---

## Mistral Large 3 (December 2025 — previous flagship; deprecated 2026-10-31, retiring 2027-04-30)

`mistral-large-2512`, 675B MoE / 41B active, 256K context, open weights. Still callable until retirement; Mistral names Large 4 as the replacement. Family rules #1, #4, #7 apply; it is the model most of this file's community-sourced persona / few-shot axes were observed on. New work: target Large 4.

---

## Mistral Medium 3.5 (April 28, 2026 — deprecated 2026-10-31, retiring 2027-04-30)

`mistral-medium-3-5` (card `mistral-medium-3-5-26-04`), 256K context, open weights under a modified MIT licence. Replacement named by Mistral: **Large 4** (not a newer Medium).

- **It does have a reasoning knob** — `reasoning_effort` `high` / `none`, with `high` recommended *"For agentic and code use cases"* (Mistral reasoning docs). Older atlas text that said "no reasoning mode" was wrong.
- No prompting guidance or system-prompt recommendation on the model card (checked 2026-07-28). Apply the family rules.

Mistral's news feed dates it 22.05.2026 ("powering remote coding agents in Vibe") while the model card says April 28 — the card date is used here.

---

## Mistral Small 4 (March 2026)

`mistral-small-2603` — the lower-cost lane of the frontier family, production-viable for high-volume / cost-sensitive workloads. `reasoning_effort` `high` / `none` (family rule #3).

### Wording-side behaviors

Same as Large 4 with one caveat: smaller models are less forgiving of vague prompts. Spell out scope and success criteria more explicitly when targeting Small 4.

---

## Ministral 3 family (3B / 8B / 14B, 2512)

The current Ministral generation (`ministral-3b-2512` etc., Apache 2.0, 256K, GA) ships **Instruct, Reasoning and Base** variants per size. These are **edge models** that straddle the frontier/small-local line. Their model cards carry the same System Prompt line as Large 4 ("define a clear environment and use case …").

| Variant | Coverage |
|---|---|
| Ministral 3-14B (Reasoning / Instruct) | This file — apply frontier-style prompting (abstract principles tolerated, few-shot helpful, longer context patterns apply) |
| Ministral 3-8B (Instruct) | This file for instruct-style usage; `small-local.md § Mistral / Ministral` for tight task-prompt regime |
| Ministral 3-3B | `small-local.md § Mistral / Ministral` — small-model regime applies |

When the user names "Ministral 3" without size, ask which one — the prompting regime diverges sharply at the 8B vs 3B boundary. The 2024 Ministral 3B / 8B API ids (`ministral-*-2410`) retired 2025-12-31.

---

## Cross-vendor rules (when Mistral frontier is one of several targets)

Mistral Large 4 fits cleanly into Claude-direction cross-vendor compromises. If a prompt already works on Claude + Kimi, it usually runs on Large 4 without changes.

| Axis | Direction on Mistral Large 4 | Cross-vendor alignment |
|---|---|---|
| Persona | helps (functional); vendor asks for environment + use case in the system prompt | matches Claude / Kimi / Gemini direction |
| Few-shot | helps (3-5 examples) | matches Claude / Kimi; only conflict is GPT-5.x reasoning |
| Step prescription | tolerated | safe across vendors |
| Reasoning lever | `reasoning_effort` low/medium/high (default high) — **no off-switch**; Ministral 3: variant choice | out-of-band like other vendors; never "don't think" in a shared prompt |
| Multi-turn history | replay the full assistant message incl. the thinking chunk | same class as Kimi K3 / DeepSeek `reasoning_content` round-trip |
| Tool guidance | OpenAI-compatible; keep the tool set minimal (vendor) | safe across vendors |
| Output format | JSON output and strict JSON schema (Large 4 card) | works alongside `json_schema` on other vendors |
| Aggressive emphasis | likely inert | matches GPT / Gemini / Kimi inert behavior |

---

## Source notes

Mistral docs are less detail-heavy on prompting than Anthropic / OpenAI / Google. Behaviors documented here come from:

- Mistral Large 4 open-weights post ([mistral.ai/news/mistral-large-4-oss](https://mistral.ai/news/mistral-large-4-oss/), datePublished 2026-10-29, read 2026-10-31 raw) — weights release, low/medium/high table ("Low … replaces 'None' reasoning mode")
- Large 4 Hugging Face card ([huggingface.co/mistralai/Mistral-Large-4-1T-A52B](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B), raw README and `chat_template.jinja`, read 2026-10-31 raw) — parameter counts, 1M, licence, Recommended Settings (reasoning levels, System Prompt and Tools lines), "strong adherence and support for system prompts", template errors on other effort values and on thinking chunk + `reasoning` field together
- Large 4 docs model card ([docs.mistral.ai/models/mistral-large-4-0](https://docs.mistral.ai/models/mistral-large-4-0.md), read 2026-10-31 raw) — API names, "Status: Public Preview", release date 2026-10-06, 1M, text + image
- Mistral reasoning docs ([docs.mistral.ai/studio/conversations/reasoning](https://docs.mistral.ai/studio/conversations/reasoning.md), read 2026-10-31 raw) — `reasoning_effort` on Small 4 / Medium 3.5 (`high`/`none`) and Large 4 (`low`/`medium`/`high`, no `none`/`minimal`/`xhigh`/`max`), preview migration note, chunk format, "always replay the full assistant message"
- Mistral models page, deprecated-and-retired table ([docs.mistral.ai/models](https://docs.mistral.ai/models.md), read 2026-10-31 raw) — Large 3 and Medium 3.5 deprecated 2026-10-31, retirement 2027-04-30, replacement Large 4; Ministral 3 (2512) lineup; Magistral native-reasoning models deprecated; 2024 Ministral ids retired 2025-12-31
- Ministral 3 card ([huggingface.co/mistralai/Ministral-3-3B-Instruct-2512](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512), read 2026-10-31 raw) — Instruct / Reasoning / Base variants, the shared System Prompt line
- Mistral's release post ([mistral.ai/news/mistral-3](https://mistral.ai/news/mistral-3)) — Large 3 facts
- Model card for `mistral-medium-3-5-26-04` ([docs.mistral.ai](https://docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04), read 2026-07-28) — facts only; the card contains no prompting guidance
- IntuitionLabs Mistral Large 3 explainer; DataCamp Mistral 3 benchmark review; Mistral's developer-guide community resources — the persona / few-shot / tone axes (community-level confidence)

Many axes are documented at lower confidence than for vendors with formal prompt-engineering guides. When confidence matters, validate against your task.
