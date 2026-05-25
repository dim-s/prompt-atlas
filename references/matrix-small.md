# Small local models — the matrix

Companion to [matrix.md](matrix.md). This one covers **task-facing prompts for small local models** (2–9B parameters, run locally via LM Studio / llama.cpp / vLLM / Ollama).

Different game from frontier agent-facing prompts: these models are usually invoked in **single-pass compile-time mode** — extract, classify, NL→DSL, score, route — with strict structured output and zero tolerance for ambiguity. The behavioral axes are different from frontier, so this is a parallel matrix, not an extension of the main one.

When reviewing a small-model prompt:
1. Identify the target model(s). When the user names a base (e.g., "gemma-4-e2b") match the closest row.
2. Read the relevant rows of both tables here.
3. Drill into [`models/small-local.md`](models/small-local.md) for vendor-family nuances the matrix can't capture (failure modes, RU/multilingual notes, fine-tune variants).
4. When citing a recommendation, **name the row × column** so the user can audit it.

When adding a new model: add a row here first. If a row needs more than 1–2 sentences per cell, push the detail into `models/small-local.md` and keep the matrix terse.

---

## How to read a cell

- **Recommendation scale** (few-shot, principles, thinking, etc.): `helps / neutral / hurts / required / forbidden`
- **Numerical**: `low / medium / high` along an axis (substitution bias, markdown emission, etc.)
- **Sweet-spot columns**: a concrete number (4 examples, 5 examples) when one is documented
- **`?`** — axis hasn't been tested for this model in our suites or in published reports; treat conservatively

A cell that says **"required"** = without this technique the model is below useful-threshold on the relevant task class. A cell that says **"forbidden"** = adding this technique causes a measurable regression we've reproduced.

---

## Table A — Structural prompt wording

How to lay out the prompt body for this model.

| Model | System role support | Critical rules position | Few-shot sweet spot | Principles tolerance (abstract rules in body) | Markdown emission default | System language (non-EN task) |
|---|---|---|---|---|---|---|
| **Gemma 3 4B** | **none** — instruct mode has only `user` / `model` roles; system goes into first user turn | END (after rule list, before output) | 4 (mirror failure modes) | low — abstract principles in middle cause cascading regressions | low | EN system unlocks +4–8pp on non-EN task input |
| **Gemma 4 e2b** | none (same as Gemma 3) | END — Google docs explicit: "critical restrictions as final line" | 4 | very low — `−7pp` regression observed when adding mid-prompt principles | low | EN system: required for non-EN task |
| **Gemma 4 e4b** | none | END | 4 | low (still hurts but ceiling higher) | low | EN system: helps, not strictly required |
| **Ministral 3B** | yes (Mistral chat template) | END (after Forbidden + named anti-patterns) | 4–5 | low — `−15pp` regression observed on mid-prompt principles in our suite | **high** — emits markdown fences spontaneously even when forbidden; **don't fight it, parser-tolerate it** | EN system: +6pp on RU input |
| **Qwen 3.5 2B** | yes (Qwen chat template); **no default system** — must be explicit | END + pre-Forbidden "CRITICAL: rules are sacred" section with named anti-patterns | **5** (one more than Gemma) | low | low | EN system: helps |
| **Qwen 3.5 9B** | yes | END | 4–5 | medium | low | EN system: neutral (model is large enough to handle non-EN well) |
| **Phi-4-mini 3.8B** | yes (`<\|system\|>...<\|end\|>` chat format) | END | ? (likely 3–5; not tested in our suites) | medium (better than Gemma family) | low | ? — Microsoft docs warn "languages other than English will experience worse performance" |
| **Llama 3.2 3B** | yes (standard Llama 3 chat template) | END | ? (likely 4–5) | medium | low | ? — RU tunes (saiga family) exist; base 3.2 weak on RU |

**Reading guide:**
- *"END"* = critical/negative rules must be the final block before the user-input area. Anything earlier risks attentional decay.
- *"none"* (system role) = Gemma instruct mode literally has no `system` role; if you pass one via API some inference engines silently merge it into the first user turn, others drop it. Verify with provider.
- *"required"* on EN system = our `nl_dsl_game` experiments saw 2-3B models gain +4–8pp on RU input when system prompt was EN, with -35-60ms latency bonus from EN tokenization.
- *"markdown emission high"* on Ministral = the model emits ```` ``` ```` fences as natural output style; building the parser to tolerate fences is cheaper than trying to suppress them (5+ prompt iterations failed to suppress in our suite).

---

## Table B — Behavioral failure surface

What the model can and can't do, and how it responds to common prompting techniques.

| Model | Implicit negation ("X but not Y") | Adjective filter in multi-rule context | Substitution bias (rule mutation) | Thinking-on impact | Native tool calling impact (vs free JSON) | Multi-pass (analyze→emit) | Self-verify same-model | Structured output format |
|---|---|---|---|---|---|---|---|---|
| **Gemma 3 4B** | medium — depends on phrasing | medium | low | n/a (no native thinking) | ? | ? | ? | JSON object (free) / json_schema OK on lmstudio |
| **Gemma 4 e2b** | **low** — fails on "but not", "except" | **low** — drops adjective in multi-rule, holds in single-rule | low | **−25pp** native thinking → unparseable | ? | **−22pp** (thinking-token leak without `--jinja`) | ? | JSON object (free); GBNF on llama.cpp |
| **Gemma 4 e4b** | medium — holds explicit "except", fails subtle "but not" | high — holds adjective in multi-rule | low | **−44pp** native thinking → unparseable | ? | −19pp (same leak as e2b) | ? | JSON object / json_schema OK on lmstudio |
| **Ministral 3B** | low — fails on "no X" preconditions, skips to easier rule | low | low | n/a (no native thinking) | n/a | **+10pp** with 2-pass analyze-then-emit (only model that wins) | **−40pp** confirmation anti-bias | JSON object; markdown-tolerant parser required |
| **Qwen 3.5 2B** | low — "every rule looks satisfied to me" | low — drops adjective like Gemma e2b | **high** — "fixes" rules ("milk" → "water_tank" because both liquids) | **−44pp** native thinking + `<think>` leak in content | ? | neutral (no gain, no loss) | ? | json_schema on lmstudio; json_object on llamacpp |
| **Qwen 3.5 9B** | medium | medium | medium | hurts (-15…-25pp) | ? | ? | ? | json_schema / json_object |
| **Phi-4-mini 3.8B** | ? | ? | ? | n/a (non-reasoning variant); `Phi-4-mini-reasoning` variant exists separately | **Microsoft documents hallucinated function names / URLs** in function-calling mode | ? | ? | JSON mode supported; tool tokens `<\|tool\|>...<\|/tool\|>` in system |
| **Llama 3.2 3B** | ? | ? | ? | n/a | JSON-based tool calling template recommended by Meta | ? | ? | JSON mode supported |

**Reading guide:**
- *"low"* / *"high"* on negation, adjective, substitution = how reliably the model handles that class. `low` = systematically fails on that class. Lift this floor with targeted few-shot examples mirroring the failure (see `techniques-small.md`).
- *"−44pp"* etc. = measured pass-rate drop on our `nl_dsl_game` / `nl_rules_farm` suites when the technique was applied. These are reproducible regressions, not theoretical concerns.
- *"hallucinated function names"* on Phi-4-mini = direct quote from [Microsoft's model card](https://huggingface.co/microsoft/Phi-4-mini-instruct). Native tool calling on 2-3B is a documented anti-pattern across vendors when the prompt is `tools-only` without few-shot demonstrations.
- *"confirmation anti-bias"* on Ministral self-verify = our finding: 17 correct answers flipped to wrong on pass-2 when prompted with "re-examine your answer". Same-model self-verification on small models is consistently negative — only **cross-model** verification works (small worker + large verifier). Backing: [arxiv:2404.17140](https://arxiv.org/pdf/2404.17140) "Small LMs Need Strong Verifiers".

---

## Cross-row patterns worth knowing

These patterns repeat across vendors and are useful when no row exists for the user's specific model.

### 1. Few-shot examples are not optional below 7B
Every 2-3B row in Table A has a sweet-spot number. The technique that consistently lifts ceiling is *demonstrations*, not *principles*. If the user has a prompt with abstract rules ("be careful with negations", "every adjective is mandatory") and no examples — that's the single highest-leverage fix.

Backing: prompt-eng literature on small LMs has converged on this. Mid-prompt principles caused `−7…−15pp` regressions on our 32-case suite. Few-shot examples placed at END caused +7pp gain with zero regressions.

### 2. Critical rules at END — not top, not middle
Google's Gemma docs say it explicitly: critical restrictions as the **final line**. Independently observed: any small model loses attentional focus on rules placed in the middle of a long prompt (especially after structured tables). The fix is positional, not lexical — moving the same words to the end works.

### 3. Thinking mode is a structural liability under ~7B
Every small row with a "Thinking on" cell shows a regression. Mechanism:
- Native thinking blocks (`<think>...</think>`) leak into `content` when not properly stripped (`--jinja` flag in llama-server, `chat_template_kwargs.enable_thinking=False` for Qwen, `reasoning_effort: none` for LM Studio — see project AGENTS.md for the multi-provider param dance)
- `max_tokens` budget gets eaten by thinking, leaving the structured output truncated → `unparseable`
- 2-3B models don't have the capacity for long CoT to actually improve answer quality — they spin

Research: [arxiv:2502.12143](https://arxiv.org/pdf/2502.12143) — "Small student models ≤3B parameters do not consistently benefit from long CoT reasoning."

### 4. Native tool calling on 2-3B without few-shot — regression
Switching from "emit JSON" to "call this tool" via `--jinja` + tools schema, *without* adding few-shot tool-call examples, regresses the model. Phi-4-mini's own card admits the failure mode (hallucinated function names). Our Gemma 4 e2b experiment: −29pp.

If the user **needs** tool mode (downstream framework constraint), pair it with 3-5 few-shot tool-call examples in system. Otherwise, free JSON with a JSON-mode constraint is more reliable on this size class.

### 5. EN system unlocks non-EN performance on 2-3B
Counterintuitive: a prompt asking the model to process Russian or German input often performs better when the **system prompt itself is English**. We verified +4–8pp on `nl_dsl_game` for Gemma 4 e2b, Mistral 3B, Qwen 3.5 2B when switching system from RU to EN, with a latency bonus from EN tokenization being denser (fewer tokens for the same semantic content).

Cross-lingual transfer is model-specific in magnitude — verify on the suite, but **default to EN system** unless the model is specifically a RU/multilingual tune (saiga, T-lite).

### 6. Same-model self-verification regresses on small models
"Re-examine your answer" / "double-check this" on a 2-3B model triggers confirmation anti-bias: the model interprets "re-examine" as "find the error" and flips correct answers to wrong. Our finding: −40pp on Ministral.

Self-verify only works with a **stronger verifier** (small worker + large checker pattern). Same-model retry on 2-3B is forbidden.

---

## When the matrix doesn't cover the model

If the user's model isn't listed:

1. **Bucket by family**: Gemma-* → Gemma row; Qwen-* → Qwen row; *-Llama / *-llama-tune → Llama row; *-Mistral / *-Mixtral / Ministral-* → Ministral row; *-Phi → Phi row.
2. **Bucket by size**: <4B → use 2-3B baseline expectations; 7-9B → expect 8B-row behavior; everything in between → "medium" on all axes.
3. **Tunes inherit base behavior** with vendor-specific overlays — most failure modes carry through. See `models/small-local.md` for known tunes (saiga, T-lite, Hermes, HORROR-Imatrix variants).
4. **Flag explicitly to the user** which row you used as proxy, so they can override if they have direct experience.
