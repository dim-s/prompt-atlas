# Model-specific wording differences — Moonshot AI Kimi family

What changes about how you should PHRASE prompts for **Kimi K3.1 Pro** (current flagship, October 2026), **K3** (previous flagship), and the K2 series still callable — **K2.7-Code** and **K2.6** (closed to new users from 31.10.2026, retiring by the end of December 2026). K2.5 and K2 are retired; their sections are in `archive/models/kimi.md`. Companion to `claude.md`, `gpt.md`, `gemini.md`, `glm.md`. This reference stays focused on wording — not API parameters, context sizes, infrastructure, or pricing; a knob is named only where a prompt line would fight it.

Kimi has its own native CLI (**Kimi Code**, `kimi.com/code`) plus OpenAI- **and Anthropic**-compatible API access through `platform.kimi.ai`. Unlike GLM, Kimi isn't typically accessed through a cross-vendor router — it runs in vendor-native or direct-API contexts.

---

## Family-wide rules (apply to all current Kimi versions)

These hold across K2.6 → K2.7-Code → K3 → K3.1 Pro (and held on the retired K2 / K2.5). Where the K3 line (K3, K3.1 Pro) or K2.7-Code changed a rule, it's marked inline.

### 1. Persona / role assignment helps — Moonshot's official guidance

Moonshot's [prompt best-practices guide](https://platform.kimi.ai/docs/guide/prompt-best-practice) is explicit: *"Assign specific roles in system messages for more accurate output. Example: Define the model as an expert in a particular domain."*

This is **opposite to GPT-5.5** (where outcome-first beats persona) and closer to Gemini 3's +5% identity-prompting boost or Claude's neutral-to-helpful persona behavior.

When porting a prompt to Kimi:
- **Keep** functional persona lines from Claude / Gemini prompts
- **Restore** persona lines stripped during GPT-5.5 tuning if you're now targeting Kimi
- Don't write meta-persona that fights model identity ("You are not an AI") — Kimi doesn't reward this

### 2. Few-shot examples help — official endorsement

The Moonshot guide explicitly recommends few-shot: *"Examples show desired style better than explicit descriptions. More efficient than describing all task permutations."*

Default 3–5 examples for format steering, similar to Claude. Unlike GPT-5.5, you do **not** need to worry about few-shot hurting reasoning quality.

### 3. Thinking is always on from K2.7-Code onward — only K2.6 still has a toggle

| Model | Thinking | Depth knob | Sampling on `platform.kimi.ai` |
|---|---|---|---|
| **K3.1 Pro**, **K3** | always on, Preserved Thinking always on | `reasoning_effort` `low` / `high` / `max`, **default `max`** | fixed: temperature 1.0, top_p 0.95 |
| **K2.7-Code** | *"forces thinking and preserve_thinking as True"* | none | fixed: temperature 1.0, top_p 0.95 |
| **K2.6** (retiring) | `thinking: {"type": "enabled"}` (default) or `"disabled"` (Instant); `"keep": "all"` retains reasoning | none | temperature 1.0 thinking / 0.6 Instant, top_p 0.95 — all fixed |

The platform's parameter reference: *"Fixed" means the parameter cannot be modified: passing any other value returns an error, so do not pass it explicitly.* (The K2.6 Hugging Face card recommended top_p 1.0 for Thinking mode — that is a self-hosting value.)

Consequences for wording:

- **Don't hardcode sampling in a Kimi prompt** — on the hosted API every current model fixes it (K2.6 per mode). Keep the body neutral.
- An Instant-mode assumption ported forward is wrong on every model above K2.6 — the model reasons and returns `reasoning_content` whatever the prompt says.
- **"Don't reason, answer immediately" / "skip the analysis" is structurally unimplementable** on K2.7-Code, K3 and K3.1 Pro (Moonshot's K3.1 Pro FAQ: *"You can't — K3.1 Pro always thinks. If the reasoning takes too long, set `reasoning_effort` to `low` to reduce the reasoning effort."*). The lever is `reasoning_effort` (K3 line) or a different model (K2.7-Code) — surface it out-of-band, never as an `[ADD]` to the prompt.
- **Pick the effort level before the conversation starts** (K3 line): *"Switching levels invalidates prefix-cache hits. Decide on the `effort` level before the conversation starts and avoid switching it mid-session."* A harness that escalates effort for the hard turn of a long session pays for the whole prefix again — a harness decision, worth naming in a review of a multi-turn Kimi setup.

### 3a. Round-trip the full assistant message (K3 line, K2.7-Code)

K3's card requires *"the complete assistant message returned by the API to be passed back to `messages` as-is — including `reasoning_content` and `tool_calls`"*; the K3.1 Pro quickstart repeats it: *"Return the complete assistant message unchanged in multi-turn conversations and tool calls."* Same class of constraint as DeepSeek V4's mandatory `reasoning_content` round-trip; K2.7-Code's forced `preserve_thinking` retains reasoning across turns for the same reason.

This is a client concern, not wording — but it's worth flagging in a review of a multi-turn Kimi prompt, because a hand-rolled message-history builder that strips reasoning breaks the loop no matter how the prompt is worded.

### 4. Use delimiters: XML, triple quotes, or section headings

The official guide encourages explicit delimiters between prompt sections — XML tags, triple quotes, or markdown headings — to separate instructions from data and clarify which sections need different processing.

This matches Claude and GPT-5.x conventions; no Kimi-specific syntax.

### 5. Step-by-step task definition for complex tasks

Moonshot's guide: *"Outline explicit steps for complex tasks. Improves model ability to follow instructions."*

This is **opposite to GPT-5.5** (which treats step prescription as noise). Kimi is closer to Claude in tolerating — and benefiting from — explicit step naming.

### 6. Output length specification

The guide recommends specifying paragraph or bullet counts (more precise than word counts). When format matters, name the structure explicitly.

### 7. Reference text with citation expectations

For fact-based queries, the guide recommends including credible source material and instructing the model to cite or state when information isn't found. This is the long-context / RAG pattern that travels well from Claude.

### 8. Standard OpenAI-compatible chat format

Kimi uses the standard OpenAI message structure: `system` / `user` / `assistant` / `tool` roles. The K2.6 HuggingFace model card shows:

```python
messages = [
    {'role': 'system', 'content': 'You are Kimi, an AI assistant created by Moonshot AI.'},
    {'role': 'user', 'content': 'your question here'},
]
```

The default system prompt baked into the platform: *"You are Kimi, an AI assistant provided by Moonshot AI. You excel at Chinese and English dialog, and provide helpful, safe, and accurate answers."* The platform also has a constraint that `"Moonshot AI"` must remain in English without translation.

When writing your own system prompt, you can replace this default entirely — the model doesn't depend on the canned identity line for behavior.

### 9. Tool calling: interleaved thinking + multi-step native

K2.6 and later support interleaved thinking and multi-step tool calls in the same response — the model can think, call a tool, see the result, think again, and call another tool, all in one turn (a design introduced with the now-retired K2 Thinking).

**Wording implication:** tools can be described as a normal toolkit without forcing the model into a "plan → act → reflect" turn structure. The model handles interleaving natively.

Built-in tools (when using the official Moonshot platform): search, code-interpreter, web-browsing. These don't need explicit tool descriptions in the prompt — they're platform-provided.

### 10. JSON Mode

Kimi exposes a "JSON Mode" for stable output shape. The exact parameter naming is OpenAI-compatible (`response_format={"type": "json_object"}`). The K2.6 HuggingFace model card doesn't elaborate JSON-schema-strict semantics. The K3.1 Pro quickstart links a Structured Output page captioned *"Use strict JSON Schema."* — that page itself was not read, so treat schema-strict semantics as `?` until confirmed against your stack.

### 11. Aggressive emphasis ("CRITICAL:", ALL-CAPS) is mostly inert

No documented overtriggering effect on Kimi (unlike Claude 4.5+). It's also not load-bearing — Kimi doesn't elevate emphasized rules' weight more than ordinary phrasing. Use sparingly for clarity, not as a behavioral lever.

### 12. Multimodal: vision on K2.6 and the K3 line (K2 / K2.5 were text-only)

K2.6 ships with a 400 M-parameter vision encoder (MoonViT); K3 and K3.1 Pro have native visual understanding (images and video). Images embed via base64 in the user message content array — standard OpenAI multimodal format (on K3.1 Pro public image URLs are not accepted — base64 or `ms://<file-id>`).

When porting a prompt written for the retired text-only K2 / K2.5, drop "this model can't see images" caveats — they're wrong on K2.6 and the K3 line (K2.7-Code's vision support is not stated on the pages read — `?`).

---

## Kimi K3.1 Pro (Moonshot, 23–24 October 2026 — current flagship)

`kimi-k3.1-pro` and `kimi-k3.1-pro-highspeed` (*"same intelligence, about 2-3× the speed"*). GA on the Kimi API, Kimi.com, Kimi Work and Kimi Code; the blog is dated 24.10 (23.10 per press — secondary). Open weights are promised, not out: *"The full model weights will be released by Nov 4."* What it is: *"a post-trained version of Kimi K3, with substantially larger post-training compute"* — same base, so this section is a delta on § Kimi K3 below.

### What changes against K3

- **Nothing on the parameter surface.** The parameter reference: *"`kimi-k3.1-pro` and `kimi-k3.1-pro-highspeed` also have the same parameter constraints as `kimi-k3`."* Thinking always on, `reasoning_effort` `low` / `high` / `max` (default `max`), sampling fixed (*"`temperature=1.0`, `top_p=0.95`, `n=1`, `presence_penalty=0`, and `frequency_penalty=0` are fixed; omit them from requests."*), full assistant message round-tripped (family rules #3, #3a). Don't pass `thinking` at all.
- **No K3.1-specific prompting guidance.** The only Moonshot prompting document is still the generic best-practices page (role, delimiters, steps, examples, length, reference text) — family rules #1–#7 carry over; no wording delta is documented.
- **Dynamic tool loading** (K3.1 Pro quickstart): *"Place a complete tool definition in a `system` message without `content`. The tool becomes available from that message onward."* — and *"Keep this message in later request history; the server does not retain it."* A harness that trims history to save tokens silently removes tools loaded this way.
- **Built-in web search is not production-ready:** *"Web search is being updated and is not recommended for production workflows in the near term."* A prompt that relies on the official search tool for fresh facts should supply reference text instead (family rule #7) until that note goes away.
- **Benchmarks: don't cite them.** The platform quickstart still carries template placeholders and its numbers differ from the blog's — neither is a sound source for a review claim.

### Migration

- **K3 → K3.1 Pro:** change the model id; prompts and parameters carry over. Re-measure at the default `max` before lowering effort.
- **K2.6 / K2.7-Code → K3.1 Pro** (forced by the K2 retirement): Moonshot — *"When migrating from K2.x to K3, remove the K2.x `thinking` configuration and use top-level `reasoning_effort` as needed."* Then apply § Migration K2.6 → K3 below; strip Instant-mode assumptions and any "answer without reasoning" line.

---

## Kimi K3 (Moonshot, July 2026 — previous flagship)

API access from July 16, 2026; open weights published July 26 (release dates from press coverage — the model card carries none). Moonshot's model list now calls it *"Kimi's previous-generation flagship"*; it is still offered.

### Headline facts

- **Architecture:** 2.8 T total parameters, **104 B active**, MoE with 896 experts (16 per token); 93 layers (69 Kimi Delta Attention + 24 Gated MLA)
- **Context:** 1,048,576 tokens (1M)
- **License:** Kimi K3 License (custom — not the Modified MIT of the K2 line)
- **Natively multimodal:** text, images and video in one model (MoonViT-V2, 401 M)
- **Quantization:** MXFP4 weights / MXFP8 activations, quantization-aware trained
- **Serving:** vLLM / SGLang / TokenSpeed; OpenAI- and Anthropic-compatible API on `platform.kimi.ai`; **Kimi Code CLI is the vendor's recommended agent framework**

### Wording behaviors specific to K3

#### Thinking is always on — depth is a parameter

*"Kimi K3 always has thinking enabled, and will return `reasoning_content`."* Reasoning effort takes `low` / `high` / `max`, defaulting to `max`. Two review consequences:

1. Any prompt line that tries to switch reasoning off, or that assumes an Instant-style fast path, is dead text — flag and strip.
2. Because the default is `max`, the *cost* complaint on K3 is usually an effort setting, not a verbose prompt. Surface `reasoning_effort: low` as the first lever before proposing a rewrite.

**Host divergence:** the same id behaves differently off Moonshot's platform. QwenCloud's API reference: *"kimi-k3 supports passing `false` to disable thinking"* for the Alibaba-supplied `kimi-k3`, while for the Moonshot-supplied `kimi/kimi-k3` there `reasoning_effort` has *"only `max` is supported."* QwenCloud's own thinking guide lists `kimi-k3` as thinking-only, so the host's two pages disagree — and it lists `kimi-k2.6` as *"Hybrid, thinking disabled by default"* (Moonshot's default is enabled). Check the host before calling a "no thinking" request a bug.

#### Sampling: fixed on the hosted API, task-shaped on self-hosted weights

On `platform.kimi.ai` the K3 series fixes temperature 1.0 and top_p 0.95 (other values return an error). The open-weight card splits its recommendations by task type — **top-p 0.95 for single-step tasks, top-p 1.0 for agentic ones** — which matters only when self-hosting. Either way, don't pin sampling in a prompt body.

#### 1M context — long-context patterns apply

Documents at top, question at bottom, quote-grounded answers, stable content first for caching. Same as Claude / Gemini / Grok 4.3 / Qwen at this window size. The jump from K2.6's 256 K means prompts that were split into chunked passes for K2.6 can often be re-validated as a single pass.

### Migration K2.6 → K3

Forward-compatible on wording (persona, few-shot, delimiters, step naming all unchanged). What breaks is code and assumptions, not phrasing:

- Thinking toggle → `reasoning_effort` parameter; `extra_body.thinking` assumptions are stale.
- Multi-turn history must round-trip the complete assistant message (family rule #3a).
- License changes from Modified MIT to a custom Kimi K3 License — relevant if the deployment relies on open-weight terms.
- Video is now first-class alongside images; "text and images only" caveats are wrong.

---

## Kimi K2.7-Code (Moonshot, June 12, 2026 — K2 series, retiring)

**Retiring:** *"starting October 31, 2026, K2 series models will no longer be available to newly registered users; existing users are not affected. We plan to retire all K2 series models by the end of December 2026"* — covers `kimi-k2.7-code` / `-highspeed` and `kimi-k2.6`. Migration target: K3.1 Pro (§ above).

The coding-specialist sibling — not a successor to K2.6 but a narrower model released alongside the line. Interesting to prompt-atlas mostly for one property: **at its release it was the first model covered here on which thinking cannot be turned off at all.**

### Headline facts

- **Architecture:** 1 T total / 32 B active, MoE with 384 experts (8 per token) — same shape as K2.6
- **Context:** 256 K
- **License:** Modified MIT
- **Thinking:** *"forces thinking and preserve_thinking as True"* — no disabled mode; reasoning is retained across turns
- **Recommended sampling:** temperature 1.0 (thinking mode), top-p 0.95
- **Efficiency:** ~30% fewer thinking tokens than K2.6 at stronger coding performance
- **Agent framework:** Kimi Code CLI; interleaved thinking + multi-step tool calls (family rule #9)

### Wording behaviors specific to K2.7-Code

#### Forced thinking — the whole class of "answer directly" instructions is void

On models that can turn thinking off, "don't overthink, answer immediately" is *weak* (inert prose where a knob belongs, `antipatterns.md` #31) or *harmful* (Opus 5 with thinking off, `antipatterns.md` #38). On K2.7-Code — as on the K3 line — it is **structurally unimplementable**: the model reasons on every turn by construction, and K2.7-Code has no effort knob either. Treat such lines as `[IMPROVE]` — delete them and, if latency is the actual complaint, note that the fix is model choice, not wording.

The same applies to prompts that try to *shape* the reasoning ("keep your analysis to two sentences") as a cost-control device — reasoning volume isn't the prompt's to set here.

#### Identity line in the vendor's example

The model card's chat example opens with `{'role': 'system', 'content': 'You are Kimi, an AI assistant created by Moonshot AI.'}` — the same canned identity as K2.6's card. It is **illustrative, not mandatory**: the card doesn't state it as required, and your own system prompt can replace it entirely (family rule #8). Don't treat it as a fixed preamble you must preserve, and don't stack a second identity line on top of it — one functional role is enough.

#### Coding specialist — scope the task, not the craft

Its gains are on coding benchmarks; general-purpose framing wastes them. Reviews of K2.7-Code prompts should push toward concrete repo scope (files, success criteria, test command) rather than toward more elaborate role text.

---

## Kimi K2.6 (Moonshot, April 20, 2026 — K2 series, retiring)

Closed to new users from 31.10.2026 and retiring by the end of December 2026 (§ Kimi K2.7-Code). Kept live because existing users still call it and because the **Agent Swarms protocol** below is documented only here.

### Headline facts

- **Architecture:** 1 T-parameter sparse MoE, 32 B active per token, 384 routed experts + 1 shared, 8 experts per token, MLA attention
- **Context:** 256 K tokens (HuggingFace model card), 98,304-token max generation length
- **License:** Modified MIT
- **INT4 native quantization**
- **Multimodal:** text + image (and experimental video via official platform)
- **Benchmarks:** Artificial Analysis Intelligence Index 54 — highest open-weights, three points behind closed-flagship frontier; ties GPT-5.5 on SWE-Bench Pro
- **Agent Swarms:** scales from K2.5's 100 sub-agents × 1,500 steps to **300 parallel sub-agents × 4,000 coordinated steps**

### Wording behaviors specific to K2.6

#### Agent Swarms protocol

Agent Swarms is K2.6's signature capability (no K3-line swarm bounds are recorded in this file — `?`; the explicit-ask wording below is the safe carry-over): the model "dynamically decomposes tasks into heterogeneous subtasks executed concurrently by self-created domain-specialized agents." It's a different operational model from single-agent prompting, designed for:
- Batch tasks (process N similar items in parallel)
- Long-form output (research compilation, multi-document synthesis)
- Large-scale search (broad parallel exploration)

**Wording implication:** the swarm capability is **explicit-ask-only**. Single-agent prompts don't accidentally trigger swarm spawning. If you want decomposition, say so:

> Decompose this into parallel sub-tasks where each can be solved independently. Spawn domain-specialized sub-agents for each. Reconcile the results into a single final answer.

If you do **not** want swarm spawning (e.g., the task is small and the overhead would dominate), the default single-agent path applies — no opt-out wording needed.

The Moonshot tech blog is explicit that "the overhead of spinning up and coordinating 300 sub-agents [is] not worth it for tasks a single agent can handle in minutes." Prompt-side, this means: don't ask for swarms on small tasks.

#### Vision-enabled persona prompts

Adding visual inputs (screenshots, design mocks) is supported. Persona prompts that target multimodal work — "You are a visual design reviewer; examine the screenshot and..." — are fine and well-aligned with K2.6's training.

### When K2.6 specific tuning helps

- Parallelizable batch work where Agent Swarms is the actual win
- Open-weight production deployments using INT4 quantization
- Long-horizon coding (`Kimi Code` CLI) with `preserve_thinking` for multi-turn reasoning continuity

### When NOT to invest in K2.6 specific tuning

- Pure single-turn classification or extraction — Instant mode does this without swarms
- Latency-critical interactive use — Thinking mode's overhead isn't worth it
- New work of any kind — the K2 series is closed to new users; target K3.1 Pro

### Migration off K2.6

Target K3.1 Pro (§ Kimi K3.1 Pro → Migration). Prompts still pinned to the retired `kimi-k2.5` (discontinued 31.08.2026) or the `kimi-k2` ids (25.05.2026) now get 404 — see `archive/models/kimi.md` for what those prompts assumed.

---

## Cross-model wording principles (Kimi side)

Mirror of the family-wide section, distilled for cross-model reviews:

- **Persona helps** (opposite to GPT-5.5; aligns with Claude / Gemini direction)
- **Few-shot helps** (3–5 examples standard; not penalized like on GPT-5.5)
- **Step-by-step prescription tolerated and sometimes encouraged** (opposite to GPT-5.5)
- **Reasoning depth is always out-of-band** — `reasoning_effort` (`low`/`high`/`max`, default `max`) on K3 / K3.1 Pro, `extra_body.thinking` on K2.6 (retiring), and **nothing at all on K2.7-Code**, where thinking is forced on; on the K3 line pick the level before the session starts (switching invalidates the prefix cache)
- **Sampling is fixed on the hosted API** (temperature 1.0 / top_p 0.95 on the K3 line and K2.7-Code; K2.6 temperature by mode) — never in the prompt body
- **Agent Swarms is explicit-ask-only** — no accidental spawning, no implicit opt-out
- **JSON Mode is OpenAI-compatible** — schema-strict semantics treat as `?` until verified
- **Multimodal on K2.6 and the K3 line** — text-only caveats inherited from the retired K2 / K2.5 are wrong
- **Aggressive emphasis mostly inert** — use for clarity, not behavior

---

## Cross-vendor rules (when Kimi is one of several targets)

If a prompt must run on Kimi **and** Claude / GPT / Gemini / GLM:

| Axis | Direction on Kimi | Cross-vendor compromise |
|---|---|---|
| Persona | helps | keep functional persona; matches Claude / Gemini direction; only conflict is GPT-5.5 where you'd strip |
| Few-shot | helps | keep 3–5 examples; only conflict is GPT-5.5 reasoning-heavy tasks |
| Step prescription | tolerated, sometimes helps | frame steps as "typical sub-tasks, choose order yourself" — works across all |
| Reasoning lever | `reasoning_effort` (K3 / K3.1 Pro) · `extra_body.thinking` (K2.6, retiring) · **none — forced on** (K2.7-Code) | parameter, not prose; don't write "think step by step", and don't write "answer without reasoning" either — it's inert where a toggle exists and impossible on K2.7-Code / K3 / K3.1 Pro |
| Output format | JSON Mode | prefer `json_schema`-compatible structured output where receiving stack supports it |
| Swarm spawning | explicit-ask | mention only if the prompt is Kimi-specific or the host supports comparable decomposition (Claude Code Task tool, Codex subagents, etc.) |

Cross-vendor wording note: Kimi's wording defaults are closer to Claude's than to GPT-5.5's. A Claude-tuned prompt usually ports to Kimi with little change; a GPT-5.5-tuned prompt may need persona and few-shot restored.

---

## Source notes

K3.1 Pro and the K2 retirement (all read 2026-10-31, raw):
- Quickstart [platform.kimi.ai/docs/guide/kimi-k3-1-pro-quickstart](https://platform.kimi.ai/docs/guide/kimi-k3-1-pro-quickstart.md) — "Important limits" (effort, fixed sampling, round-trip, vision input, web search), the "can't turn off" FAQ, dynamic tool loading, the Nov 4 weights promise. Its benchmark paragraph still has template placeholders — not used.
- Reasoning effort [platform.kimi.ai/docs/guide/use-reasoning-effort](https://platform.kimi.ai/docs/guide/use-reasoning-effort.md) — ladder, default, the K2.x → K3 migration line.
- Parameter reference [platform.kimi.ai/docs/api/models-overview](https://platform.kimi.ai/docs/api/models-overview.md) — "same parameter constraints as `kimi-k3`", the fixed-sampling table for K3 / K2.7-Code / K2.6, the prefix-cache warning on switching effort.
- Thinking guide [platform.kimi.ai/docs/guide/use-thinking-models](https://platform.kimi.ai/docs/guide/use-thinking-models.md) — Preserved Thinking always on for K3.1 Pro; don't pass `thinking`.
- Model list [platform.kimi.ai/docs/models](https://platform.kimi.ai/docs/models.md) — K3.1 Pro / Highspeed descriptions, K3 as "previous-generation flagship", the K2-series notice (closed to new users 31.10, retirement by end of December 2026), K2.5 discontinued 31.08, K2 25.05.
- Blog [kimi.com/blog/kimi-k3-1-pro](https://www.kimi.com/blog/kimi-k3-1-pro) — "post-trained version of Kimi K3", dated 24.10; the 23.10 launch date is from press (secondary, not read in full).
- QwenCloud API reference [docs.qwencloud.com/api-reference/chat/openai-chat](https://docs.qwencloud.com/api-reference/chat/openai-chat.md) — the `kimi-k3` host divergence.

K3 and K2.7-Code facts come from their HuggingFace model cards ([moonshotai/Kimi-K3](https://huggingface.co/moonshotai/Kimi-K3), [moonshotai/Kimi-K2.7-Code](https://huggingface.co/moonshotai/Kimi-K2.7-Code), read 2026-07-28). Neither card carries a release date; the July 16 / July 26 (K3 API / weights) and June 12 (K2.7-Code) dates are from press coverage. **Moonshot has published no model-specific prompting guide for either** — the family-wide best-practices guide is still the only prompting document, so K3/K2.7-Code wording advice here is derived from documented model *behavior* (forced thinking, effort levels, sampling), not from vendor prompting guidance.

The official Moonshot prompt best-practices guide ([platform.kimi.ai/docs/guide/prompt-best-practice](https://platform.kimi.ai/docs/guide/prompt-best-practice)) is text-light on K2.6 specifics — it documents the general best practices that apply to the family. K2.6-specific behavior (Agent Swarms scale, vision encoder, two-mode temperature split) comes from the HuggingFace model card and the Moonshot tech blog. Where this file says "Moonshot documents X," the citation is one of those two sources.

Schema-strict JSON mode semantics and certain agentic-loop behaviors are marked `?` — they haven't been independently verified against documentation, and prompt-atlas should default to conservative wording (don't assume strict-schema behavior without testing) when those axes matter for a review.
