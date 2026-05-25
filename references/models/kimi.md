# Model-specific wording differences — Moonshot AI Kimi family

What changes about how you should PHRASE prompts for Kimi K2.6 (and legacy K2.5 / K2). Companion to `claude.md`, `gpt.md`, `gemini.md`, `glm.md`. This reference stays focused on wording — not API parameters, infrastructure, or pricing.

Kimi has its own native CLI (**Kimi Code**, `kimi.com/code`) plus standard OpenAI-compatible API access through `platform.moonshot.ai`. Unlike GLM, Kimi isn't typically accessed through a cross-vendor router — it runs in vendor-native or direct-API contexts.

---

## Family-wide rules (apply to all current Kimi versions)

These hold across K2 → K2.5 → K2.6.

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

### 3. Two-mode design: Thinking vs Instant

K2.6 (and K2 family) ships with two operational modes:

| Mode | Trigger | Temperature | top_p | Use case |
|---|---|---|---|---|
| **Thinking** | `extra_body={'thinking': {'type': 'enabled'}}` (default) | 1.0 | 1.0 | Reasoning, agentic loops, complex tasks |
| **Instant** | `extra_body={'thinking': {'type': 'disabled'}}` | 0.6 | 0.95 | Quick lookups, classification, latency-critical |

Multi-turn flows with retained reasoning use `extra_body={'thinking': {'type': 'enabled', 'keep': 'all'}}` (or `preserve_thinking: True` on vLLM/SGLang chat_template_kwargs).

**Wording implication:** the temperature split is real and documented. Don't hardcode a single temperature into a Kimi-targeted prompt body. The body should be neutral; let the API config pick the mode.

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

K2.6 uses the standard OpenAI message structure: `system` / `user` / `assistant` / `tool` roles. The HuggingFace model card shows:

```python
messages = [
    {'role': 'system', 'content': 'You are Kimi, an AI assistant created by Moonshot AI.'},
    {'role': 'user', 'content': 'your question here'},
]
```

The default system prompt baked into the platform: *"You are Kimi, an AI assistant provided by Moonshot AI. You excel at Chinese and English dialog, and provide helpful, safe, and accurate answers."* The platform also has a constraint that `"Moonshot AI"` must remain in English without translation.

When writing your own system prompt, you can replace this default entirely — the model doesn't depend on the canned identity line for behavior.

### 9. Tool calling: interleaved thinking + multi-step native

K2.6 supports interleaved thinking and multi-step tool calls in the same response — the model can think, call a tool, see the result, think again, and call another tool, all in one turn. This is the same design as K2 Thinking, formalized as a stable feature in K2.6.

**Wording implication:** tools can be described as a normal toolkit without forcing the model into a "plan → act → reflect" turn structure. The model handles interleaving natively.

Built-in tools (when using the official Moonshot platform): search, code-interpreter, web-browsing. These don't need explicit tool descriptions in the prompt — they're platform-provided.

### 10. JSON Mode

Kimi K2 series exposes a "JSON Mode" for stable output shape. The exact parameter naming is OpenAI-compatible (`response_format={"type": "json_object"}`). The K2.6 HuggingFace model card doesn't elaborate JSON-schema-strict semantics; treat schema-strict mode as `?` until confirmed against your stack.

### 11. Aggressive emphasis ("CRITICAL:", ALL-CAPS) is mostly inert

No documented overtriggering effect on Kimi (unlike Claude 4.5+). It's also not load-bearing — Kimi doesn't elevate emphasized rules' weight more than ordinary phrasing. Use sparingly for clarity, not as a behavioral lever.

### 12. Multimodal: vision via MoonViT (K2.6 adds vision; K2 was text-only)

K2.6 ships with a 400 M-parameter vision encoder (MoonViT). Images embed via base64 in the user message content array — standard OpenAI multimodal format. Video input is experimental and only on the official platform.

When porting a K2 (pure-text) prompt to K2.6, drop "this model can't see images" caveats — they're now wrong.

---

## Kimi K2.6 (Moonshot, April 20, 2026 — current frontier)

### Headline facts

- **Architecture:** 1 T-parameter sparse MoE, 32 B active per token, 384 routed experts + 1 shared, 8 experts per token, MLA attention
- **Context:** 256 K tokens (HuggingFace model card), 98,304-token max generation length
- **License:** Modified MIT
- **INT4 native quantization** (same method as K2 Thinking)
- **Multimodal:** text + image (and experimental video via official platform)
- **Benchmarks:** Artificial Analysis Intelligence Index 54 — highest open-weights, three points behind closed-flagship frontier; ties GPT-5.5 on SWE-Bench Pro
- **Agent Swarms:** scales from K2.5's 100 sub-agents × 1,500 steps to **300 parallel sub-agents × 4,000 coordinated steps**

### Wording behaviors specific to K2.6

#### Agent Swarms protocol

Agent Swarms is K2.6's signature capability: the model "dynamically decomposes tasks into heterogeneous subtasks executed concurrently by self-created domain-specialized agents." It's a different operational model from single-agent prompting, designed for:
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
- Tasks the smaller K2.5 already solves well — don't migrate without measured gains

### Migration from K2.5 → K2.6

Forward-compatible. Trim "this is a text-only model" assumptions in prompts that disabled image-handling tools. Review swarm-decomposition prompts if you used K2.5's 100-agent / 1500-step bounds — K2.6 raises both.

**Important:** Moonshot announced K2 (and K2 series) **discontinues May 25, 2026**. Prompts pinned to `kimi-k2-0905-preview` etc. need migration to K2.6 endpoints by that date.

---

## Kimi K2.5 (Moonshot, March 2026 — predecessor)

### Headline facts

- **Pure text** (no vision)
- Smaller Agent Swarms scope: 100 sub-agents × 1,500 steps
- Same temperature split (1.0 thinking / 0.6 instant) as K2.6
- 256 K context

### When K2.5 specific tuning is still useful

- Existing production stacks where you've measured prompt behavior and don't want to migrate yet
- Cost-sensitive deployments where K2.5 is "good enough"

Prompts written for K2.5 generally port forward to K2.6 with no rewrites. The reverse (K2.6 → K2.5) requires trimming swarm scale and removing image handling.

---

## Kimi K2 (legacy, retires May 25, 2026)

Earliest production-frontier Kimi model. Most current tooling has migrated past it. The official `platform.kimi.ai` quickstart still references K2 endpoints (`kimi-k2-0905-preview`, `kimi-k2-turbo-preview`, `kimi-k2-thinking`, `kimi-k2-thinking-turbo`), which **all retire May 25, 2026**.

If you encounter a prompt pinned to K2 endpoints in a current review, treat the retirement date as a forcing function and flag the prompt for migration to K2.6.

---

## Cross-model wording principles (Kimi side)

Mirror of the family-wide section, distilled for cross-model reviews:

- **Persona helps** (opposite to GPT-5.5; aligns with Claude / Gemini direction)
- **Few-shot helps** (3–5 examples standard; not penalized like on GPT-5.5)
- **Step-by-step prescription tolerated and sometimes encouraged** (opposite to GPT-5.5)
- **Two-mode design (Thinking/Instant) is the load-bearing knob** — set via `extra_body`, not prose
- **Temperature split** between modes is documented and meaningful — don't override
- **Agent Swarms is explicit-ask-only** — no accidental spawning, no implicit opt-out
- **JSON Mode is OpenAI-compatible** — schema-strict semantics treat as `?` until verified
- **Multimodal in K2.6 only** — K2 / K2.5 prompts that assume text-only are wrong on K2.6
- **Aggressive emphasis mostly inert** — use for clarity, not behavior

---

## Cross-vendor rules (when Kimi is one of several targets)

If a prompt must run on Kimi **and** Claude / GPT / Gemini / GLM:

| Axis | Direction on Kimi | Cross-vendor compromise |
|---|---|---|
| Persona | helps | keep functional persona; matches Claude / Gemini direction; only conflict is GPT-5.5 where you'd strip |
| Few-shot | helps | keep 3–5 examples; only conflict is GPT-5.5 reasoning-heavy tasks |
| Step prescription | tolerated, sometimes helps | frame steps as "typical sub-tasks, choose order yourself" — works across all |
| Reasoning lever | `extra_body.thinking` | parameter, not prose; don't write "think step by step" |
| Output format | JSON Mode | prefer `json_schema`-compatible structured output where receiving stack supports it |
| Swarm spawning | explicit-ask | mention only if the prompt is Kimi-specific or the host supports comparable decomposition (Claude Code Task tool, Codex subagents, etc.) |

Cross-vendor wording note: Kimi's wording defaults are closer to Claude's than to GPT-5.5's. A Claude-tuned prompt usually ports to Kimi with little change; a GPT-5.5-tuned prompt may need persona and few-shot restored.

---

## Source notes

The official Moonshot prompt best-practices guide ([platform.kimi.ai/docs/guide/prompt-best-practice](https://platform.kimi.ai/docs/guide/prompt-best-practice)) is text-light on K2.6 specifics — it documents the general best practices that apply to the family. K2.6-specific behavior (Agent Swarms scale, vision encoder, two-mode temperature split) comes from the HuggingFace model card and the Moonshot tech blog. Where this file says "Moonshot documents X," the citation is one of those two sources.

Schema-strict JSON mode semantics and certain agentic-loop behaviors are marked `?` — they haven't been independently verified against documentation, and prompt-atlas should default to conservative wording (don't assume strict-schema behavior without testing) when those axes matter for a review.
