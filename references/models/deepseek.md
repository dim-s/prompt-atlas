# Model-specific wording differences — DeepSeek family

What changes about how you should PHRASE prompts for **DeepSeek V4.1 Flash** (`deepseek-flash`, build 1017) and **V4-Pro** (`deepseek-v4-pro`, build 0813). Retired and legacy DeepSeek models (V4-Flash, V4-Flash-Vision-Exp, V3.2, R1) are in `archive/models/deepseek.md`. Companion to `claude.md`, `gpt.md`, `gemini.md`, `kimi.md`, `glm.md`, `qwen-frontier.md`. This reference stays focused on wording — not API parameters, context sizes, or pricing, except where one changes prompt design.

DeepSeek is the most opinionated of the current frontier vendors in **where to put instructions** — and that opinion is opposite to most other vendors. This file leads with that rule because the cost of getting it wrong is high, and because two measurements have since narrowed it.

---

## Version lineage (2026) — which model an id serves today

DeepSeek ships silent upgrades behind stable ids, so a pinned id is not a pinned model. Read this table before trusting any "measured on" note.

| Date | API id / weights repo | What it is | Served by today |
|---|---|---|---|
| 2026-04-24 | `deepseek-v4-pro`, `deepseek-v4-flash`; HF `DeepSeek-V4-Pro` / `-V4-Flash` (+ `-Base`, 04-22) | V4 preview (Pro and Flash) | Pro id → V4-Pro-0813; Flash id → V4.1-Flash-1017 |
| 2026-06-27 | HF `DeepSeek-V4-Pro-DSpark` / `-V4-Flash-DSpark` | *"is **not** a new model. It is the same checkpoint with an additional speculative decoding module attached"* | not a prompt target |
| 2026-07-31 | `deepseek-v4-flash`; HF `DeepSeek-V4-Flash-0731` | official Flash: *"keeps the same model architecture and size as DeepSeek-V4-Flash-Preview, and was only re-post-trained"* | retired → id served by V4.1-Flash-1017 (archive) |
| 2026-08-13 | `deepseek-v4-pro`; HF `DeepSeek-V4-Pro-0813` | V4-Pro GA; effort ladder `low`/`high`/`max` introduced for Pro and Flash | **current** — § V4-Pro |
| 2026-08-21 | `deepseek-v4-flash-vision-exp`; HF repo 2026-08-31 | experimental vision sibling of V4-Flash | retired → id served by V4.1-Flash-1017 (archive) |
| 2026-09-10 | `deepseek-flash`; HF `DeepSeek-V4.1-Flash` | V4.1 Flash — new architecture, native multimodal | superseded by 1017 behind the same id |
| 2026-10-17 | `deepseek-flash` (+ both legacy Flash ids); HF `DeepSeek-V4.1-Flash-1017` | *"a minor update to DeepSeek-V4.1-Flash"*, shorter outputs | **current** — § V4.1 Flash |
| 2025 | V3.1 (08-21), V3.1-Terminus (09-22), V3.2-Exp (09-29), **V3.2 (2025-12-01)** + V3.2-Speciale; R1-0528 (2025-05-28) | pre-V4 line | `deepseek-chat` / `deepseek-reasoner` were pointed at V4-Flash on 2026-04-24 and announced for discontinuation 2026-07-24; the current pricing page lists neither (archive) |

---

## Family-wide rules (current DeepSeek: V4-Pro, V4.1 Flash)

Most of these come from **V4-era practitioner guides** and were not restated by DeepSeek for V4.1. Where a rule was measured or documented on a specific model, it says which.

### 1. Core instructions live in the USER prompt, not the system prompt — DeepSeek-specific

Practitioner guidance for V4 (citing the documented R1 behavior) is to **place core instructions in the user prompt**, keep the system prompt brief, and treat the user prompt as the binding contract. No current DeepSeek page states this: the changelog, the thinking-mode guide and both V4.1 Flash cards (read 2026-10-31) say nothing about system-prompt adherence either way.

**This is opposite to most other vendors.** Claude, GPT, Gemini, Kimi and frontier Qwen all expect the bulk of instruction in the system prompt.

| Other vendors | DeepSeek (practitioner pattern) |
|---|---|
| **System prompt:** "You are a senior backend engineer. Follow these rules: [50 lines of constraints, examples, formatting, edge cases]. Use these tools when..." | **System prompt:** "You are a senior backend engineer." |
|  | **User prompt:** "[Task description.] Constraints: [...]. Examples: [...]. Format: [...]. Use these tools when..." |

Wording mitigation when porting Claude / GPT prompts to DeepSeek:

- **Move the bulk of the system content into the user message.** Keep a single-line functional role in the system slot.
- **For role-play and thinking-mode work specifically:** put instructions at the **end of the first user message** — per V4 practitioner guides, the position where instruction-following is most stable.
- **Don't fight this with workarounds** like "in the system prompt, the rules are: X" referenced from the user prompt.

Quantitative claim from V4 prompting guides: **system prompt overuse causes ~85% of common DeepSeek V4 errors.** It has **failed to reproduce twice**, on two unrelated task classes — both on **V4-Flash-0731**, which is now retired and two Flash builds behind the model `deepseek-flash` serves (field notes below). Apply the rule where you can cheaply comply; do not re-architect a working system-prompt agent before measuring it on the model you actually target.

> **Measured on `deepseek-v4-flash:0731` (archived model) — not re-run on V4.1 Flash or V4-Pro.** Both notes below are kept as measurements; they do not transfer to V4.1 as results, only as the reason to measure first.

> **Field note (2026-08-21) — second non-reproduction, and the practical one: Claude-authored agent bodies port to V4-Flash unchanged.** A 12.7 KB
> subagent body (Russian copy-editing methodology: ~23 numbered transformation rules in three blocks,
> each with before/after pairs, plus register detection and a hard "high-stakes texts get minimal
> intervention" boundary) was written for `claude-opus-5` and mirrored into the Pi harness **as the
> system prompt, verbatim**, on `ollama/deepseek-v4-flash:0731` with `--thinking max`. Four fixtures,
> each targeting a different failure class: apply-the-method, resist-invention under a "make it more
> interesting" prompt, high-stakes medical dosage instruction, and API documentation (over-editing trap).
>
> V4-Flash executed the system-side methodology **more faithfully than `claude-sonnet-5` did** on the
> same four fixtures across three transports. On the invention trap it added nothing and said plainly
> that the text could not be enlivened without fabricating; on the medical instruction it made one
> clarity edit and named the register's function; sonnet-5, by contrast, returned one fixture unedited
> claiming "block A found nothing" while missing a copular filler verb the same body's rule A3 names
> explicitly. The one flaw found on V4-Flash was a *mild over-application* (rhythm edit inside a
> technical doc) — the opposite of the under-weighted-system-methodology failure rule #1 predicts.
>
> Practical consequence worth stating for reviewers: **an agent body authored against Opus/Sonnet
> conventions can be pointed at V4-Flash without moving the bulk to the user turn.** Reviewing such a
> port, do not open with "move the instructions to the user prompt" — measure first. `--thinking` level
> mattered far more than instruction placement here: `high` rewrote impersonal medical prescriptions
> into imperatives (a real safety-relevant over-application), `max` did not.
>
> **Limits:** one body, four fixtures, one run each, single channel (Ollama Cloud subscription route,
> not the direct `api.deepseek.com`); no A/B against the same body served from the user turn, so this
> shows "system-side works", not "system-side is equal or better". Judged by one human pass against a
> rubric, on binary markers (removed the filler verb or not; rewrote the prescription or not).

> **Field note (2026-08-19) — did not reproduce on V4-Flash under a system-only agent body.** A 10.2 KB
> agent body consisting entirely of system-side methodology (route-read a 170 KB third-party document
> by its table of contents → read only the matching sections → query HTTP endpoints → judge) was run as
> the system prompt on `deepseek-v4-flash` (channel: `opencode/deepseek-v4-flash-free`). The task was
> chosen to favour the failure mode above: it requires search in a large unfamiliar document plus a
> conclusion about ABSENCE. Two runs — one with inherited MCP tools, one with an explicit
> `read,grep,bash` allowlist and a `mktemp -d` cwd. Both executed the system methodology precisely: the
> trace shows `grep -n '^## '` for the ToC, then `read` at exactly the two matching offsets out of
> twenty-four sections, then the endpoint calls, then a refusal citing the sections by name. No
> fabricated negative, no mis-scoped search.
>
> **Limits, stated plainly:** one agent, one task class, two runs — this is a non-reproduction on one
> workload, not a refutation of the ~85% figure; the channel was the free route, not the direct
> `deepseek/deepseek-v4-flash` API, and the two were not compared. DeepSeek's official changelog
> (V4-Pro 2026-08-13, V4-Flash 2026-07-31; read 2026-08-19) says **nothing** about system-prompt
> adherence in either direction, so neither this note nor the rule above can currently be settled from
> the vendor. Rule #1 stands; treat this as a datapoint that it may be narrower than stated. Detail in
> the maintainers' reconciliation log of 2026-08-19 (not published).

**Agentic failure mode (practitioner reports, V4).** A tool-using DeepSeek agent that carries its rules and search methodology in the system prompt tends to **fabricate confident false negatives** on absence questions ("X doesn't exist anywhere" for something that does) and mis-scope its searches. Moving the bulk to the user turn (system kept to a one-line role) is reported to fix it on the same model and tools. Symptom → diagnosis: a confident "doesn't exist anywhere" from a DeepSeek agent → suspect system-prompt overuse and verify before trusting the negative. (The 08-19 note above is the one atlas measurement of this exact failure class, and it did not reproduce.)

### 2. Brief system prompt + structured user prompt = the canonical V4 pattern

```
SYSTEM: You are a senior architect specializing in distributed systems.

USER: <context>
  <file path="src/auth/session.py" relevance="primary">
    [code]
  </file>
  <file path="docs/auth-flow.md" relevance="reference">
    [excerpt]
  </file>
</context>
<instruction>
Find the race condition. Output as JSON: {issue: string, root_cause: string, fix: string}.
</instruction>
```

- **System:** one-line functional role
- **User `<context>`:** structured data with explicit `relevance` tags
- **User `<instruction>`:** the task, with output shape demanded in prose **even when** `response_format` is set

Practitioner-guide benchmarks report ~92% task success for this XML-tagged structure vs ~45% for unstructured dumps (V4-era; not vendor-published).

### 3. Reasoning history: round-trip it with tools; without tools it is dropped

Thinking-mode guide (read 2026-10-31): *"for requests carrying the `tools` parameter, the `reasoning_content` must be fully passed back to the API in all subsequent requests — even for turns where the model did not perform a tool call. If your code does not correctly pass back `reasoning_content`, the API will return a 400 error."* Without `tools`, *"`reasoning_content` does not need to be passed back; even if passed to the API, it will be ignored and will not be concatenated into the context."* The open-weight V4.1 encoding does the same (`drop_thinking`: without tools, *"reasoning before the latest user request is stripped"*; with tools it is kept).

R1 and V3.2 **rejected** `reasoning_content` on input — the migration trap is in `archive/models/deepseek.md`.

Wording-side: the round-trip is a client concern, don't reference `reasoning_content` in prose. One derived consequence: in a tool-less multi-turn chat, a prompt that asks the model to "build on your earlier reasoning" points at text that is no longer in context — restate the conclusion in the visible turn instead.

### 4. Tool calls in thinking mode

V4 and V4.1 support tool calls while reasoning (R1 forced a choice — `archive/models/deepseek.md`). Prompts that instruct "first reason about the task, then in a separate response call the tool" can be simplified.

The open-weight V4.1 encoding injects its own tool block into the system prompt, including *"If thinking_mode is enabled (triggered by <think>), you MUST output your complete reasoning inside <think>...</think> BEFORE any tool calls or final response."* and *"You MUST strictly follow the above defined tool name and parameter schemas to invoke tool calls."* Don't restate either in your own prompt; and the vendor's ALL-CAPS here is template text, not emphasis advice.

### 5. Effort-model thinking lever: `thinking` toggle + effort strings

Since the V4-Pro GA update (2026-08-13) reasoning depth is an **effort model** for Pro and Flash; the old `thinking: "off" | "high" | "max"` set is obsolete. Surfaces (thinking-mode guide, read 2026-10-31 — the table spans columns, so read it carefully):

| Format | Thinking toggle | Effort control |
|---|---|---|
| OpenAI Chat Completions | `{"thinking": {"type": "enabled/disabled"}}` (via `extra_body` in the OpenAI SDK) | `reasoning_effort`: `low`/`high`/`max` |
| Anthropic | `{"thinking": {"type": "enabled/disabled"}}` | `output_config.effort`: `low`/`high`/`max` |
| Responses API | `{"reasoning": {"effort": "none/low/high/max"}}` — *"none disables thinking mode"* | same field |

**Thinking is on by default at effort `high`.** Accepted aliases map as: `minimal` → `low`; `low` → `low`; `medium` → `high`; `high` → `high`; `xhigh` → `high`; `max` → `max`; `ultra` → `max`. So there are three real levels; `medium` and `xhigh` silently become `high`.

**Open weights: the integer and the strings are one scale.** The V4.1 encoding README: *"Reasoning effort is a numeric budget (1–100)… String aliases map as follows: `"low"` → 50, `"high"` → 75, `"max"` → 100. The default is `"high"` (75)."* The prefix is rendered only in thinking mode and only once, at the start of the conversation; intermediate integers *"may be used to elicit interpolated reasoning behavior"*. For the hosted API the documented values remain the strings above; an integer there is undocumented (`?`) — don't advise it.

Wording-side: don't write "think step by step", and don't ask the model to lower its own reasoning in prose — set the parameter. A "don't think / answer immediately" line is at most a reduced-reasoning request; the real off-switches are `thinking.type: "disabled"` (Chat Completions / Anthropic) and `reasoning.effort: "none"` (Responses).

### 6. Demand JSON in the user prompt even with `response_format` set

Practitioner guidance (V4): even with `response_format={"type": "json_object"}`, the user prompt must still explicitly demand JSON. The combination of API parameter + prose demand is the load-bearing pattern; V4 guides estimate missing-JSON-instruction as ~60% of structured-output errors.

```
... Output strictly as JSON matching this shape: {field1: ..., field2: ...}. No prose outside the JSON.
```

Costs little on other vendors and is load-bearing here — a safe cross-vendor default.

### 7. Slightly more literal than Claude

Practitioner guidance (V4; not restated for V4.1): DeepSeek follows instructions more literally than Claude and does not infer intent. When porting a Claude-tuned prompt:
- Spell out scope: "modify only the function `validateInput`" instead of "modify the validation logic"
- Spell out boundaries: "do not refactor anything outside this function" — honored without overtriggering
- Spell out success criteria: "the existing tests pass plus a new regression test you add"

### 8. Cache-hit economics reward stable front matter

On the current pricing page a cache-hit input token costs **~1/50 of a cache miss on `deepseek-flash` and ~1/30 on `deepseek-v4-pro`** (read 2026-10-31; peak and off-peak keep the ratio). That ratio, not the absolute price, is what changes prompt design: front-loading static documentation and few-shot examples is cheap *if* that block stays byte-stable across calls. Don't ask "is this too long?" — ask "is this stable enough to be cached?"

### 9. 1M context nominal; reasoning reported to degrade past ~500K

Practitioner reports on V4: reasoning quality degrades past ~500K tokens, retrieval stays viable to ~750K. Not re-measured on V4.1 (whose card claims a much smaller KV-cache footprint — an infrastructure claim, not a quality one). Wording: keep reasoning tasks under ~500K; for both at scale, retrieve first, then reason on the retrieved chunk.

### 10. No standard Hugging Face chat template — use DeepSeek's encoding

V4 and V4.1 ship no Jinja template; V4.1 provides `encoding/encoding.py` plus the `deepseek-recipe` Rust library. V4.1 also changed the tool-call markup (`<｜DSML｜ calls>` with a leading space; V4 used `<｜DSML｜tool_calls>`). When a prompt is served through generic tooling, confirm it uses DeepSeek-aware encoding for the right generation.

---

## DeepSeek V4.1 Flash (2026-09-10; build 1017 since 2026-10-17 — current Flash, `deepseek-flash`)

Released 2026-09-10 as a new architecture; upgraded in place to **DeepSeek-V4.1-Flash-1017** on 2026-10-17 (*"The API calling method and pricing remain unchanged — simply use `deepseek-flash` to access the latest model."*). The legacy ids are routed here: *"The legacy names `deepseek-v4-flash` and `deepseek-v4-flash-vision-exp` are still accepted, but the corresponding models have been retired, their requests are served by the DeepSeek-V4.1-Flash-1017 model"*. A harness pinning either old id is running this model. No V4.1 Pro has been released.

### Headline facts

- **Architecture (card):** 552B backbone, 8B activated per token on prefill / 16B on decode; 1M context; **native multimodal** (*"The model natively processes images and text"*); MIT, open weights. Vision ✓ on `deepseek-flash`, not supported on `deepseek-v4-pro` (pricing page).
- **Effort:** rule #5 — hosted strings with the alias table; open-weight 1–100 with `low`/`high`/`max` = 50/75/100, default 75.
- **Sampling:** hosted thinking mode ignores `temperature` / `presence_penalty` / `frequency_penalty` without error; on `deepseek-flash` `top_p` works only in thinking mode, range 0.95–1.0 (below 0.95 is treated as 0.95), fixed at 1.0 in non-thinking mode. The 1017 card recommends `temperature` 1.0, **`top_p` 1.0** (the 09-10 card said *"0.95 or 1.0"*) and `max_tokens` ≥ 256K.
- **New prompt-format surfaces in the open-weight encoding:** *"Mid-conversation system messages are supported via the `<｜System｜>` token"*, and a `latest_reminder` role (*"Latest reminder (date, locale, etc.)"*). Tool results are merged into the preceding user message as `<tool_result>` blocks. Whether the hosted API exposes either role is undocumented — `?`.
- **Images:** the encoding's examples carry images inside user content blocks (`<｜deepseek_image｜>` placeholders in prompt order). The Vision-Exp rule *"images in system or assistant messages return 400"* (archive) was documented for the `-vision-exp` id and was not re-read for `deepseek-flash` — placement restrictions on V4.1 are `?`. Keeping task images in user turns is safe either way.

### Wording behaviors

- **1017 is terser by design — carried-over brevity instructions may now overcorrect (hypothesis, re-measure).** The vendor: *"we also reduced the model's task runtime and output length"*; the release post: *"Stronger benchmark performance, faster task completion, and shorter outputs than the previous version"*. Same pattern the atlas records for GPT-5.6 and Gemini 3.6 Flash: a global "be brief / no preamble" written against a longer predecessor stacks on an already-shorter model. Express length as a task requirement ("at most 5 bullets"), not a disposition, and re-check outputs that must stay complete (reports, diffs, migrations). DeepSeek published no measurement of how much shorter, so this is a review flag, not a rule.
- **Turn-scoped instructions have a native slot on open weights.** Self-hosted, a mid-conversation system message or a `latest_reminder` (date, locale) can carry per-turn context instead of editing the first system message. On the hosted API, put such context in the newest user turn until the roles are documented there.
- **Family rules #1–#10 are inherited, not re-measured.** The field notes under rule #1 were measured on V4-Flash-0731 (archived); keep "measure first", don't carry the non-reproduction over as a V4.1 result.
- No first-party prompting guide; no other documented wording delta versus V4-Flash. Don't invent one.

---

## DeepSeek V4-Pro (2026-04-24 preview; GA build 0813 since 2026-08-13 — current Pro, `deepseek-v4-pro`)

DeepSeek continues V4-Pro after 2026-09-14 (*"continue providing API services for DeepSeek V4 Pro"*); the id serves `DeepSeek-V4-Pro-0813`.

### Headline facts

- **Architecture:** 1.6T total / 49B activated, MoE (reported figure); 1M context; **text-only** (vision lives in `deepseek-flash`)
- **GA update (2026-08-13):** agentic gains (Terminal Bench 2.1 87.9, DeepSWE 62.7 from 12.8 on the preview — vendor figures); native OpenAI Responses API surface with Codex configuration; the effort ladder of rule #5
- **Sampling differs from Flash:** in thinking mode `top_p` is ignored (*"fixed at 0.95 when tool calling is enabled, otherwise 1.0"*); in non-thinking mode it takes effect

### Wording behaviors specific to V4-Pro

- **All family rules** (user-prompt priority, XML-tagged context, JSON-in-prose) are the V4-era baseline here; none was re-measured by the atlas on 0813.
- **Tool-use gap vs the closed flagships narrowed but did not close** with GA (Artificial Analysis Index 53 at GA — secondary). For long-horizon agentic loops where tool reliability dominates, the closed frontier may still be the better target; for scoped reasoning + tool use V4-Pro is competitive at much lower cost.

### When V4-Pro specific tuning helps

- Complex reasoning workloads where cached-prompt economics are the actual win (rule #8)
- Tightly scoped coding (specific function, specific bug) — literalism is a strength
- Long-context retrieval (under ~500K for reasoning, ~750K for retrieval — rule #9)

### When NOT to invest

- Long-horizon autonomous agents — the closed frontier leads on this axis
- Visual / multimodal input — not supported on this id; target `deepseek-flash`

---

## Cross-model wording principles (DeepSeek side)

- **User-prompt priority** (opposite to every other vendor) — practitioner rule, twice not reproduced on V4-Flash-0731; measure on the current model
- **Brief system prompt** — system overuse is the #1 reported failure mode
- **XML-tagged context with `relevance` attrs** — practitioner 92% vs 45%
- **Demand JSON in prose AND set `response_format`** — both, not either
- **Effort-model thinking lever** — `thinking` toggle + effort strings, three real levels (`low`/`high`/`max`; `minimal`/`medium`/`xhigh`/`ultra` are aliases); off via `thinking.type: disabled` or Responses `effort: none`; open weights 1–100 with 50/75/100
- **Tool calls work in thinking mode**
- **`reasoning_content` round-trip with tools** (400 if missing); without tools it is dropped
- **Slightly more literal than Claude** — spell scope, boundaries, success criteria
- **Cache-hit economics** reward byte-stable front matter
- **1M nominal, ~500K reasoning ceiling** (V4 reports) — split retrieval and reasoning stages
- **No standard HF Jinja templates** — DeepSeek-specific encoding, generation-specific tool markup
- **V4.1 Flash 1017 writes shorter** — re-check carried-over brevity lines

---

## Cross-vendor rules (when DeepSeek is one of several targets)

DeepSeek's user-prompt-priority rule is the **load-bearing cross-vendor question**. If a single prompt must run on DeepSeek **and** Claude / GPT / Gemini / Kimi / GLM / frontier Qwen:

### Option A — write for DeepSeek, accept slight overhead on others

Move the bulk of instruction into the user message. Other vendors read it from there with little penalty. Cost: a lean system prompt may look "incomplete" in tools that surface system content to humans.

### Option B — write for the others, accept possibly worse DeepSeek behavior

Bulk in system. Acceptable if DeepSeek is a tertiary target. The ~85% failure attribution is a practitioner figure that did not reproduce in two atlas measurements — measure before paying for Option A.

### Option C — maintain a DeepSeek-specific override

If the artifact is `AGENTS.md`, ship an `AGENTS.deepseek.md` override that replaces the system bulk with a brief role and moves the rest into a template injected into the user message.

**Recommended default:** Option A unless the user has a reason to favor the others; for an existing, working system-prompt agent, measure first.

### Cross-vendor wording table addendum (DeepSeek-specific)

| Axis | Direction on DeepSeek | Cross-vendor compromise |
|---|---|---|
| Instruction placement | **user prompt** (system stays brief) — practitioner rule, measure | move bulk to user; system is one-line role |
| XML context structure | helps strongly (practitioner 92% vs 45%) | use XML; works on Claude / GPT / Gemini / Kimi too |
| JSON-in-prose demand | required (with `response_format`) | demand JSON in prose universally |
| Cache-hit front-loading | helps strongly (cache hit ~1/30–1/50 of a miss) | works on all vendors, less dramatically |
| Tool-use in thinking | works | safe |
| `reasoning_content` round-trip | mandatory with tools | client-library concern, not wording |
| Length instructions | V4.1 Flash 1017 is terser than its predecessor | state length as a task requirement |

---

## Source notes

- DeepSeek API changelog, [api-docs.deepseek.com/updates](https://api-docs.deepseek.com/updates) — read 2026-10-31 (raw): the 2026-10-17 V4.1-Flash-1017 entry (*"reduced the model's task runtime and output length"*, id unchanged), 2026-09-10 V4.1 Flash (native multimodal; V4 Flash and Vision-Exp retired, ids routed), 2026-08-21 Vision-Exp, 2026-08-13 V4-Pro GA (effort `low`/`high`/`max`, Responses API), 2026-07-31 V4-Flash-0731 (*"only re-post-trained"*), 2026-04-24 V4 (`deepseek-chat`/`deepseek-reasoner` discontinuation 2026-07-24), 2025-12-01 V3.2, 2025-09-29 V3.2-Exp, 2025-09-22 V3.1-Terminus, 2025-08-21 V3.1, 2025-05-28 R1-0528. Earlier reads 2026-08-19 / 08-29 / 09-24 (the 09-24 read was a WebFetch summary)
- DeepSeek news post [news261017](https://api-docs.deepseek.com/news/news261017) — read 2026-10-31 (raw): *"Stronger benchmark performance, faster task completion, and shorter outputs than the previous version"*
- [Models & Pricing](https://api-docs.deepseek.com/quick_start/pricing) — read 2026-10-31 (raw): MODEL VERSION `DeepSeek-V4.1-Flash-1017` / `DeepSeek-V4-Pro-0813`, Vision ✓ only on `deepseek-flash`, legacy-id routing note, cache-hit vs cache-miss rates (ratio used in rule #8); neither `deepseek-chat` nor `deepseek-reasoner` listed
- [Thinking-mode guide](https://api-docs.deepseek.com/guides/thinking_mode) — read 2026-10-31 (raw HTML; the toggle row spans the OpenAI and Anthropic columns, and the Responses cell `reasoning.effort` none/low/high/max spans both rows — earlier atlas text had the Anthropic and Responses fields swapped): default `high`, alias table incl. `minimal` → low and `ultra` → max, `top_p` behavior per model, `reasoning_content` pass-back rules (400 with tools)
- HF [DeepSeek-V4.1-Flash-1017 card](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017/raw/main/README.md) and [encoding README](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017/raw/main/encoding/README.md) — read 2026-10-31 (raw): minor update, sampling `temperature` 1.0 / `top_p` 1.0 / `max_tokens` ≥ 256K, effort aliases 50/75/100 default 75, mid-conversation `<｜System｜>`, `latest_reminder` role, `drop_thinking`, DSML leading-space change, the injected tool block's MUST lines, images in user content blocks
- HF [DeepSeek-V4.1-Flash card](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash/raw/main/README.md) — read 2026-10-31 (raw): 552B / 8B prefill / 16B decode, native multimodal, integer 1–100 effort, `top_p` *"0.95 or 1.0"*
- HF [DeepSeek-V4-Flash-DSpark card](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-DSpark/raw/main/README.md) — read 2026-10-31 (raw): *"is **not** a new model"*; HF API list for `deepseek-ai` (raw, 2026-10-31) — repo creation dates in the lineage table
- DeepSeek's vision guide ([api-docs.deepseek.com/guides/vision](https://api-docs.deepseek.com/guides/vision)) — last read 2026-08-29 for the Vision-Exp id; **not re-read for V4.1** (hence the `?` on image placement)
- Datanorth's V4-Pro-0813 coverage (datanorth.ai, 2026-08-14) — AA Index 53 (secondary)
- DeepSeek V4 practitioner guides (deepseekai.guide, skywork.ai), the CodersEra V4 vs V3.2 comparison, the BentoML "Complete Guide to DeepSeek Models" — the source of rules #1, #2, #6, #7, #9 and their percentages

The strongest documented claims are the **`reasoning_content` round-trip** and the **effort surface** (both vendor pages). **User-prompt priority** is practitioner-sourced and twice not reproduced on an archived build — treat it as a measure-first default. Cache economics are vendor-priced; the ~500K reasoning ceiling is a V4-era practitioner report.

DeepSeek publishes no official prompting guide for V4 or V4.1 (checked against the changelog, news and both V4.1 cards on 2026-10-31) — guidance comes from model cards, the encoding README, release notes and practitioner guides. Treat this file as the synthesis it is.
