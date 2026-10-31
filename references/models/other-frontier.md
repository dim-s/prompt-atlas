# Model-specific wording differences — other frontier families (MiniMax, Xiaomi MiMo, StepFun)

What changes about how you should PHRASE prompts for **MiniMax M3.1-Flash**, **Xiaomi MiMo-V2.6 Pro / Flash** and **StepFun Step 5 Preview**. Companion to the other model files in this directory. This reference stays focused on wording — not API parameters, context sizes, or pricing, except where one changes prompt design.

**These are families without a vendor prompting guide.** Each vendor documents its model (card, chat template, API pages) and at most a generic prompting page; none says how prompting this model differs from others. So this file records the documented facts that touch wording — reasoning knobs, what the template injects, what must be round-tripped — and leaves every behavioral axis (literalism, persona, few-shot, emphasis, verbosity, instruction placement) `?`. Same treatment as `tencent.md`. A review that claims a model-specific behavioral delta for these families beyond the facts below is fabricating.

---

## Shared notes

### 1. Apply universal defaults; the knobs are the only documented levers

For all three: write to the cross-vendor defaults (`_universal.md`) — bulk in the system prompt, explicit scope, structure over emphasis, tool guidance in tool descriptions — and put reasoning depth in the API parameter, never in prose. What differs between them is the knob: MiniMax has a five-level effort ladder and no off-switch, MiMo has an on/off toggle and no ladder, StepFun has a three-level ladder and no off-switch.

### 2. Reasoning must be round-tripped, and the field name differs

MiniMax and MiMo return reasoning in `reasoning_content` and require it back in history on tool-calling turns (MiMo: 400 otherwise). StepFun returns it in a field named **`reasoning`**. A harness ported from DeepSeek / Qwen that copies `reasoning_content` will miss StepFun's trace. Client concern, not wording — but name it when a multi-turn agent prompt "forgets" its plan on one vendor only.

---

## MiniMax M3.1-Flash (weights 2026-10-17; API 2026-10-19)

GA on the MiniMax API, M Plan and MiniMax Code; supersedes the September M3.1-Flash-Preview (M Plan only). MiniMax-M3 remains available; a Pro variant is announced as coming *"later"*.

### Headline facts

- **Architecture:** ~428B total / 23B active MoE; text, image and video in; 1M context; open weights under the **MiniMax Community License** (not OSI): free for non-commercial use; commercial use requires displaying *"Built with MiniMax M3.1-Flash"*, plus a one-time notice to MiniMax — or prior written authorization above US$20M yearly revenue
- **Thinking cannot be disabled (hosted):** *"Thinking cannot be turned off. Sending `thinking: {"type": "disabled"}` or `effort: "none"` returns `400`"* — *"To reduce the latency and token usage of thinking, lower the `effort` level rather than trying to disable thinking."*
- **Effort (hosted):** `low`, `medium`, `high`, `xhigh`, `max`; *"For `MiniMax-M3.1-Flash`, omitting `effort` defaults to `max`."* Field per protocol: Anthropic-compatible `output_config.effort` (the vendor's recommended surface), OpenAI-compatible `reasoning_effort`, Responses `reasoning.effort`
- **Tool history:** *"the entire `response_message`, including `reasoning_content`, must be kept in the message history"*
- **Open weights:** recommended `temperature=1.0`, `top_p=0.95`; effort set in the chat template, **no default there** (the `<effort>` tag is rendered only when passed — unlike hosted `max`)

### What the chat template injects (open weights)

- **No system prompt → an identity line:** *"Your model version is MiniMax-M3.1, developed by MiniMax. Knowledge cutoff: January 2026. …"* A system prompt with content replaces it.
- **Always appended:** a `<thinking_instructions>` block ending *"Current thinking mode: adaptive. You are encouraged to think for complex decision-making, multi-step reasoning, or when analyzing function/tool results."*
- Optional `current_date` / `current_location` fields on the system message; a separate `developer` role (default *"You are a helpful assistant."*); reasoning rendered for every past assistant turn.

### Wording consequences (from the facts above)

- **Always ship a system prompt on self-hosted deployments** — otherwise the model is told its identity and a January 2026 cutoff by the template. If date matters, use the `current_date` field rather than prose.
- **Don't add "think before you answer / reason about tool results" lines** — the template already says it, every turn.
- **Hosted default is `max`** — the most expensive level. Set effort explicitly per workload; a "keep it short / don't overthink" line cannot switch thinking off.
- **`?`:** persona, few-shot, literalism, emphasis, verbosity, instruction placement.

### MiniMax M2.5 / M2.7 — retiring 2026-11-18

*"October 19, 2026: These models will no longer be available to new users. November 18, 2026: These models will be retired."* On M Plan / Token Plan, *"all requests to M2.5 and M2.7 models … will automatically be routed to **M3.1 Flash**"* since 19.10 — a plan-routed prompt "on M2.x" is already running M3.1-Flash. (The same FAQ page's "Retired Models" table already lists M2.7 / M2.5 as Retired — the page is internally inconsistent; the notice dates are the schedule.) The atlas has no M2.x sections.

---

## Xiaomi MiMo-V2.6 Pro / Flash (2026-09-22)

Ids `mimo-v2.6-pro`, `mimo-v2.6-flash`, `mimo-v2.6-pro-ultraspeed`. The previous generation retired: `mimo-v2.5-pro` → `mimo-v2.6-pro` and `mimo-v2.5` → `mimo-v2.6-flash`, old names error since **2026-10-21 10:00 (UTC+8)**. The 9B distill (`MiMo-V2.6-Distill-Qwen-9B`) is a Class 2 model — see `small-local.md`.

### Headline facts

- **Architecture:** Pro 1.02T total / 42B active, Flash 309B / 15B; 1M context; text, image, video, audio in; MIT, open weights
- **Thinking toggle (hosted):** *"Set the `thinking.type` parameter in the request to control deep thinking: `enabled` to enable deep thinking or `disabled` to disable deep thinking."* Enabled by default on all three v2.6 ids. **No effort ladder** documented.
- **Thinking toggle (open weights):** `enable_thinking=False` in the chat template emits an empty `<think></think>`; thinking is on by default
- **Sampling:** in deep thinking *"do not support custom `temperature` and `top_p` parameters"* — forced to `1.0` / `0.95`; the cards recommend the same values
- **Tool history:** with tool calls in history, *"must completely pass back the `reasoning_content` field, otherwise the API will return a 400 error"*; *"If historical `reasoning_content` is missing, the model's context will be incomplete, which may result in decreased instruction following and increased hallucinations."*
- **MOPD checkpoints** (HF, 27.09) target a named V2.6 failure: *"the model would sometimes issue the same or highly similar tool calls repeatedly, consuming time and context without making progress. This checkpoint mitigates it."* Whether the hosted ids serve the MOPD weights is not stated — `?`.

### Wording consequences

- **The vendor's own system-prompt shape is identity + date + cutoff:** *"You are MiMo, an AI assistant developed by Xiaomi. Today is date: Tuesday, December 16, 2025. Your knowledge cutoff date is December 2024."* Supplying the current date in the system prompt follows the vendor example; nothing else about system-prompt content is documented.
- **Depth is binary** — `thinking.type` / `enable_thinking`; there is no intermediate level to approximate with prose.
- **Repeated tool calls on V2.6 are a vendor-acknowledged model issue**, fixed by a checkpoint, not by wording — if a review sees looping tool calls, check which weights serve the id before adding anti-repetition prose. Whether such prose helps is `?`.
- **`?`:** persona, few-shot, literalism, emphasis, verbosity, instruction placement.

---

## StepFun Step 5 Preview (weights 2026-09-20; `step-5-preview`)

StepFun's agentic flagship, in preview. The API release date is not on the vendor pages read; the HF repo was created 2026-09-20 (`-NVFP4` on 14.10).

### Headline facts

- **Architecture:** 600B total / 27B active MoE; text, image and video in; 1M context; max output 64k (*"`max_tokens` defaults to `INF`"*); weights under the StepFun Model License 1.0 (*"an Apache-2.0-Style license, with additional attribution and commercial-license conditions"*)
- **Effort:** *"Supports `low`, `medium`, and `high`; use `reasoning_effort` with the Chat Completions API and `output_config.effort` with the Messages API"*. The reasoning guide labels `medium` *"Default recommendation, suitable for general reasoning and multi-step tasks"*; whether `medium` is also what the API applies when effort is omitted is not stated — `?`. Published benchmark results *"use the **High** reasoning setting"*.
- **Always thinks:** the template's generation prompt always opens `<think>`; there is no off-switch in the template, and none on the API pages read
- **Reasoning field:** *"they include a `reasoning` field in the output to display the model's thinking process"* — not `reasoning_content`
- **Template:** a passed effort is prepended inside the system turn as `Reasoning: {effort}`; `preserve_thinking` defaults to true; a non-first `system` message named `observation` renders as an `observation` role
- **Sampling (card):** *"Recommended inference settings: `temperature=1.0`, `top_p=0.95`, `top_k=0`."*

### Wording consequences

- **Set effort explicitly** — the vendor recommends `medium`, benchmarks at `high`, and the omitted-effort default is undocumented. A prose "reason briefly" is not a substitute.
- **StepFun's prompting page is generic** (clear instructions, delimiters, few-shot, *"Give the model time to think"*, Role/Goal/Constraints/Output/Workflow templates) — not Step 5-specific; it gives no delta to apply.
- **`?`:** persona, few-shot, literalism, emphasis, verbosity, instruction placement.

---

## Cross-vendor rules (when one of these is among several targets)

| Axis | MiniMax M3.1-Flash | MiMo-V2.6 Pro / Flash | Step 5 Preview | Cross-vendor alignment |
|---|---|---|---|---|
| Reasoning depth | `effort` low…max, hosted default `max`; no off-switch | `thinking.type` on/off (default on); no ladder | `reasoning_effort` low/medium/high; always thinks | put depth in the parameter; no "think harder / less" prose |
| Reasoning in history | `reasoning_content`, keep whole message | `reasoning_content`, 400 if missing with tools | field `reasoning`; template preserves | client concern — check the field name per vendor |
| System prompt | template injects identity if absent | vendor example: identity + date + cutoff | effort line rendered into system turn | always ship a system prompt; date via field or system line |
| Instruction placement, persona, few-shot, literalism, emphasis | `?` | `?` | `?` | let the documented vendors in the set decide |

---

## Source notes

- MiniMax: [text-generation guide](https://platform.minimax.io/docs/guides/text-generation.md) — read 2026-10-31 (raw): effort levels, default `max`, field per protocol, no-disable 400, `reasoning_content`; [M3 function-call guide](https://platform.minimax.io/docs/guides/text-m3-function-call.md) (raw): keep the entire `response_message`; [release notes](https://platform.minimax.io/docs/release-notes/models.md) (raw): «Oct. 19, 2026 — MiniMax M3.1 Flash»; [models FAQ](https://platform.minimax.io/docs/faq/models.md) (raw): M2.5 / M2.7 schedule and plan routing; [blog](https://www.minimax.io/blog/minimax-m31f) (raw): Pro *"later"*; HF [MiniMax-M3.1-Flash card](https://huggingface.co/MiniMaxAI/MiniMax-M3.1-Flash/raw/main/README.md) and [chat_template.jinja](https://huggingface.co/MiniMaxAI/MiniMax-M3.1-Flash/raw/main/chat_template.jinja) (raw): 428B / 23B, sampling, identity line, `<thinking_instructions>`, `<effort>` only when passed, `developer` role; [LICENSE](https://huggingface.co/MiniMaxAI/MiniMax-M3.1-Flash/raw/main/LICENSE) (raw, 2026-10-31): non-commercial grant, attribution line, notice vs authorization at US$20M. HF repo created 2026-10-17 (HF API, raw)
- Xiaomi MiMo: [deep-thinking guide](https://mimo.mi.com/static/docs/quick-start/usage-guide/text-generation/deep-thinking.md) — read 2026-10-31 (raw): `thinking.type`, defaults, forced sampling, `reasoning_content` 400, vendor system-prompt example; [model release log](https://mimo.mi.com/static/docs/updates/model.md) (raw): 2026-09-22 V2.6 series, three ids; [deprecation page](https://mimo.mi.com/static/docs/updates/deprecate.md) (raw): V2.5 → V2.6 mapping, 2026-10-21 10:00 UTC+8; HF [MiMo-V2.6-Pro-MOPD](https://huggingface.co/XiaomiMiMo/MiMo-V2.6-Pro-MOPD/raw/main/README.md) / Flash-MOPD cards and chat templates (raw): MIT, sizes, modalities, sampling, `enable_thinking`, tool-call repetition note
- StepFun: [Step 5 Preview model page](https://platform.stepfun.ai/docs/en/guides/models/step-5-preview.md) — read 2026-10-31 (raw): effort levels and fields, 64k output, `max_tokens` INF; [reasoning guide](https://platform.stepfun.ai/docs/en/guides/developer/reasoning.md) (raw): `medium` *"Default recommendation"*, `reasoning` field; [prompt guide](https://platform.stepfun.ai/docs/en/guides/developer/prompt.md) (raw): generic; HF [Step-5-Preview card](https://huggingface.co/stepfun-ai/Step-5-Preview/raw/main/README.md) and [chat_template.jinja](https://huggingface.co/stepfun-ai/Step-5-Preview/raw/main/chat_template.jinja) (raw): 600B / 27B, sampling, High-setting benchmarks, licence, `Reasoning:` line, always-open `<think>`, `preserve_thinking`, `observation`; HF repo dates via HF API (raw)
- **Not read:** MiniMax pricing and the M3.1-Flash Responses-API page; any StepFun API release announcement; whether hosted MiMo ids serve the MOPD weights
- **No model-specific prompting guide from any of the three vendors as of 2026-10-31** — every unlisted behavioral axis is deliberately `?`
