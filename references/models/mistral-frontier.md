# Model-specific wording differences — Mistral frontier family

What changes about how you should PHRASE prompts for **Mistral Large 3** (Dec 2025, 675B MoE / 41B active) and the still-evolving Mistral frontier ecosystem (**Mistral Medium 3.5** in April 2026, Mistral Small 4 in March 2026, Ministral 3 series for edge use).

**This file covers Mistral frontier only.** Smaller Ministral 3B / 8B / 14B edge variants and older Ministral / Mistral 7B-class variants live in `small-local.md`. Don't confuse them — the prompting regime differs (frontier tolerates abstract principles; small Ministral regresses on the same wording).

---

## Family-wide rules

### 1. Functional persona helps — close to Claude / Kimi direction

Mistral docs and community guides converge on **functional persona being helpful**, similar to Claude's direction (neutral-to-helpful) and Kimi's (officially endorsed). Not as load-bearing as Gemini's +5% identity boost, but no penalty like on GPT-5.5.

When porting from a Claude prompt, persona lines usually travel cleanly to Mistral Large 3 without changes.

### 2. Standard OpenAI-compatible chat format

Mistral uses the standard `system` / `user` / `assistant` / `tool` role structure. Tool calling is OpenAI-compatible (`tools` array, function definitions). No surprise format requirements.

### 3. Reasoning variants are a separate model, not a toggle

Unlike Claude (`effort`), Gemini (`thinking_level`), or DeepSeek (`thinking: max`), Mistral splits **reasoning and instruct as separate model variants**. The Ministral 3 family ships base / instruct / **reasoning** variants per size.

**Wording implication:**
- If the workload needs explicit reasoning, the lever is **model selection**, not a parameter
- Don't write "think step by step" — same family rule as other current frontier vendors
- For reasoning-variant prompts, the model already reasons by default; tune for outcome wording instead

### 4. Few-shot helps — no GPT-5.5-style penalty

Mistral tolerates and benefits from few-shot examples for format steering. 3–5 examples is a reasonable default, similar to Claude. Don't strip few-shot blocks when porting from Claude prompts.

### 5. 256K context (Mistral Large 3)

Long-context patterns apply: documents top, question bottom, "Based on the above..." anchor. Same as Claude / GPT mid-tier context behavior.

### 6. Apache 2.0 licensing (Ministral) / open weights (Large 3)

Mistral is the European open-weights frontier story. Production deployments may run self-hosted, on Mistral's API, or through partner clouds (NVIDIA partnership announced for the model family).

**Wording-side this doesn't change anything** — same model, multiple hosts. But it matters for prompt portability: prompts that work on the API surface should also work on self-hosted vLLM/SGLang/llama.cpp deployments without changes.

### 7. Tone — direct, similar to GPT-5.5 / Mistral's historical character

Default tone is task-oriented and terse. Mistral doesn't have Claude's validation-forward warmth or Gemini's structured-but-concise. Ask explicitly for warmth or extended explanation if needed.

### 8. Image understanding in instruct/reasoning variants

Ministral 3 family variants (and Large 3) have image-understanding capabilities. When porting older Mistral 7B-era prompts that assumed text-only, audit for new vision-handling cases.

---

## Mistral Large 3 (December 2025 — current frontier)

### Headline facts

- **675 B MoE total, 41 B activated** per forward pass
- **256K context**
- **Open weights**
- **NVIDIA partnership** for accelerated deployment

### Wording-side behaviors

- Apply family rules above
- Closer to Claude direction than GPT-5.5 — keep persona, keep few-shot, tolerate step-by-step
- For reasoning-heavy work, pick the reasoning variant rather than scaffolding the instruct variant with prose

---

## Mistral Medium 3.5 (April 28, 2026)

`mistral-medium-3-5-26-04` — the mid-tier the family previously lacked, sitting between Small 4 and Large 3.

### Headline facts

- **256K context**, open weights under a Modified MIT license
- **$1.50 / $7.50** per M tokens (in / out)
- Vendor positioning: *"Our frontier-class multimodal model optimized for agentic and coding use cases"*
- Supports function calling, agents & conversations, built-in tools, structured outputs, prefix, predicted outputs, FIM, OCR/document processing, batching

### Wording-side behaviors

**Nothing model-specific is documented.** The model card carries no prompting guidance, no system-prompt recommendation, no reasoning mode and no temperature advice — this was checked directly, and the absence is the finding, not an omission in this file. Apply the family-wide rules above and the Large 3 defaults (functional persona, few-shot fine, step naming tolerated, long-context patterns at 256K).

Left explicit so a future coverage pass doesn't re-derive it: *if* Mistral publishes prompting guidance for the Medium tier, that's the trigger to expand this section — the model's existence alone doesn't warrant more than the facts above.

Mistral's news feed dates it 22.05.2026 ("powering remote coding agents in Vibe") while the model card says April 28 — the card date is used here.

---

## Mistral Small 4 (March 2026)

### Headline facts

- Smaller-footprint sibling — same family character at lower compute cost
- Production-viable for high-volume / cost-sensitive workloads
- Generally treated as Large 3's lower-cost lane within the frontier family

### Wording-side behaviors

Same as Large 3 with one caveat: smaller models are less forgiving of vague prompts. Spell out scope and success criteria more explicitly when targeting Small 4 vs Large 3.

---

## Ministral 3 family (3B / 8B / 14B)

These are **edge models** that straddle the frontier/small-local line. The 14B reasoning variant has frontier-adjacent capability; the 3B base is solidly small-local territory.

| Variant | Coverage |
|---|---|
| Ministral 3-14B-reasoning | This file — apply frontier-style prompting (abstract principles tolerated, few-shot helpful, longer context patterns apply) |
| Ministral 3-8B-instruct | This file for instruct-style usage; `small-local.md § Mistral / Ministral` for tight task-prompt regime |
| Ministral 3-3B-base | `small-local.md § Mistral / Ministral` — small-model regime applies |

When the user names "Ministral 3" without size, ask which one — the prompting regime diverges sharply at the 8B vs 3B boundary.

---

## Cross-vendor rules (when Mistral frontier is one of several targets)

Mistral Large 3 fits cleanly into Claude-direction cross-vendor compromises. If a prompt already works on Claude + Kimi, it usually runs on Mistral Large 3 without changes.

| Axis | Direction on Mistral Large 3 | Cross-vendor alignment |
|---|---|---|
| Persona | helps (functional) | matches Claude / Kimi / Gemini direction |
| Few-shot | helps (3-5 examples) | matches Claude / Kimi; only conflict is GPT-5.5 reasoning |
| Step prescription | tolerated | safe across vendors |
| Reasoning lever | model variant selection, not parameter | doesn't conflict with API knobs on other vendors |
| Tool guidance | OpenAI-compatible | safe across vendors |
| Output format | `json_object` standard | works alongside `json_schema` on other vendors |
| Aggressive emphasis | likely inert | matches GPT / Gemini / Kimi inert behavior |

---

## Source notes

Mistral docs are less detail-heavy on prompting than Anthropic / OpenAI / Google. Behaviors documented here come from:

- Mistral's own release post ([mistral.ai/news/mistral-3](https://mistral.ai/news/mistral-3))
- Model card for `mistral-medium-3-5-26-04` ([docs.mistral.ai](https://docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04), read 2026-07-28) — facts only; the card contains no prompting guidance
- IntuitionLabs Mistral Large 3 explainer
- DataCamp Mistral 3 benchmark review
- Mistral's developer-guide community resources

Many axes are documented at lower confidence than for vendors with formal prompt-engineering guides. When confidence matters, validate against your task.
