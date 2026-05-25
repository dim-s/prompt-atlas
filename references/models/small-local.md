# Small local model families — vendor deltas

Companion to [matrix-small.md](../matrix-small.md). Family-level notes that don't fit the matrix cells: known patho failures, fine-tune variants, vendor-specific quirks, and recommended starting prompt for each family.

When the user's model is listed here — read its section. When not listed — match to the closest family by name pattern (see `matrix-small.md § When the matrix doesn't cover the model`) and read that family's notes.

---

## Google Gemma family

### Lineup (2026)
- **Gemma 3 1B / 4B / 12B / 27B** — text-only, dense decoder. 128k context (4B+).
- **Gemma 4 e2b / e4b** — "edge" variants, designed for on-device. Smaller param count, larger activations than 2B/4B equivalents (e2b ≈ 2B params + extra capacity through E-mixing).
- **Gemma 3n / 4n** — multimodal variants (image+text input).
- **Notable fine-tunes**: HORROR-Imatrix (gemma-3-4b, horror-tuned), saiga_gemma3_12b (Russian instruct tune), TrevorJS uncensored variants.

### License
- **Gemma 4: Apache 2.0** (Google switched from custom Gemma license for v4). Clean for commercial deployment.
- Gemma 3 and earlier: custom Gemma Terms of Use (restrictive on redistribution but permissive for inference).

### The defining quirk: no system role

Gemma instruction-tuned models have only `user` and `model` roles. System-level instructions go into the **first user turn**. Different inference engines handle the API-level `system` field inconsistently — some merge into user turn, some drop, some throw.

**Always** structure for Gemma as:
```python
messages = [
    {"role": "user", "content": f"{system_prompt}\n\n{actual_input}"},
]
```

Or hide the merge in your client wrapper. Don't trust API-level `system` to survive a provider switch.

### Failure surface (verified on our suites)

| Failure class | Gemma 3 4B | Gemma 4 e2b | Gemma 4 e4b |
|---|---|---|---|
| Implicit negation ("but not", "except") | medium | **low** | medium |
| Adjective filter in multi-rule context | medium | **low** | high |
| Substitution bias | low | low | low |
| Markdown emission | low | low | low |
| Thinking-on impact | n/a (no native) | **−25pp** if forced | **−44pp** if forced |
| Multi-pass (analyze→emit) | ? | **−22pp** (token leak) | −19pp |

### Recommendations

**For task-facing prompts on Gemma family:**
1. Use the skeleton in `techniques-small.md § Skeleton` — Forbidden + 4 examples at END
2. Skip the system role; fold instructions into first user turn
3. **Don't enable thinking** on any Gemma variant for compile-time tasks. The Gemma 4 native thinking blocks leak heavily.
4. EN system unlocks +4-8pp on non-EN tasks (we measured on `nl_dsl_game`)
5. For Gemma 4 e2b specifically: keep example count at exactly 4; 5+ doesn't help and starts hurting

**Quant choice:**
- Gemma 4 4-bit MLX has a known bug with PLE — garbage output after fine-tuning. Use bf16 for training, q8 for final packaging.
- For Gemma 3 4B: **Q4_K_M beats Q8_0** on our 58-case nl_dsl_game suite (91% vs 88%, reproduced 3 runs). Counterintuitive — Q4 reshape distributions in a way that happens to help our task. Test before assuming Q8 is "safer".

### Variant: HORROR-Imatrix (gemma-3-4b)
- Horror-tuned with sensory-invasion-style few-shot
- Standard Gemma 3 4B failure surface, but loss of some general-purpose performance for non-horror prompts
- Our `horror_entity` suite: 100% pass on v2 system (NOT-DO + few-shot)

### Variant: saiga_gemma3_12b
- Russian instruct-tuned by IlyaGusev. Heavy Russian corpus shift.
- **EN-system anti-pattern (#6) doesn't apply here** — same-language Russian system can match or beat EN system on Russian tasks
- Otherwise behaves like Gemma 3 12B for structural axes

---

## Mistral / Ministral family

### Lineup (2026)
- **Mistral 7B v0.3** — original instruct, still useful baseline
- **Ministral 3B / 8B** (2024 release, called "Mistral 3B/8B" in some places). Apache 2.0 since 2024 release.
- **Mixtral 8x7B / 8x22B** — MoE, not in "small" category but worth knowing the family pattern
- **Mistral Nemo 12B** (Mistral + NVIDIA collab)
- **Codestral** — code-tuned variants

### License
- **Ministral 3B (`Ministral-3B-Instruct-2412`): Apache 2.0** — clean commercial use
- Caveat: earlier Mistral Research License existed for some 8B variants — verify exact build before commercial deployment
- Codestral: separate non-production license

### The defining quirk: markdown emission

Mistral family emits ` ```json ... ``` ` fences as natural output style. Suppressing it takes 5+ iterations and still leaks ~10% of the time. **Don't fight it — parser-tolerate it.** See `techniques-small.md § Markdown-tolerant output`.

This isn't a bug, it's a stylistic preference baked into instruction tuning. Mistral's docs describe JSON output as "Reliable, flexible and practical" — the model treats fences as part of "reliable formatting" courtesy.

### Failure surface (verified, Ministral 3B)

| Failure class | Ministral 3B |
|---|---|
| Implicit negation | low — skips "no X" preconditions, falls through to next rule |
| Adjective filter | low |
| Substitution bias | low |
| Markdown emission | **high** — emits fences habitually |
| Self-verify same-model | **−40pp** confirmation anti-bias |
| 2-pass analyze→emit | **+10pp** — the only model in our suite where multi-pass actually wins |

### Recommendations

**For task-facing prompts on Mistral family:**
1. Use the skeleton + **markdown-tolerant output paragraph** in the Output section
2. Add **named anti-patterns block** ("Critical anti-patterns — DO NOT DO THESE") with named modes: Substitution / Inferring outside rules / Skipping negations / Dropping adjectives — see `techniques-small.md § Anti-pattern named block`
3. 4-5 examples at END
4. **Try 2-pass (analyze → emit) for tasks with complex rules** — Ministral is the one small model where this consistently wins
5. **Never use same-model self-verify** — biggest single regression source on this family

### When Ministral is the right pick
- Tasks where you can pay 3-4× latency for 2-pass → +10pp gain
- Strong system-instruction adherence with explicit anti-pattern naming
- Apache 2.0 license matters for commercial deployment

### When to avoid
- Tight latency budgets (single-pass max)
- Tasks heavy on negation and "except" clauses — Ministral skips "no X" preconditions reliably

---

## Alibaba Qwen family

### Lineup (2026)
- **Qwen 3.5 0.6B / 2B / 4B / 8B / 9B / 32B / 235B** — current generation, mixed dense + MoE
- **Qwen 3.5 Coder** variants (code-tuned)
- **Qwen 2.5** still in active production deployments; behaves differently than 3.5 on system prompts

### License
- **Qwen 3.5: Apache 2.0** — clean commercial
- Qwen 2.5 and earlier: Tongyi Qianwen License (mostly permissive but requires acknowledgment)

### The defining quirk: substitution bias + thinking-mode complexity

**Substitution bias** — Qwen 3.5 2B (and to a lesser extent 4B) systematically "fixes" rules for the player. If a rule says "water with milk" and milk isn't in the inventory whitelist, Qwen 2B swaps `milk` for `water_tank` because "they're both liquids" — without error.

**Thinking-mode complexity** — Qwen 3 family has native thinking. Controlled by:
- `enable_thinking=True/False` parameter in `tokenizer.apply_chat_template`
- Soft switches `<|think_on|>` / `<|think_off|>` anywhere in system or user prompt — but **only work when `enable_thinking=True`**; ignored when `enable_thinking=False` (because the empty `<think>\n\n</think>` block is already inserted)
- Default behavior: thinking enabled

For inference engines: Qwen 3.5 has **no default system prompt** (unlike Qwen 2.5 which ships with "You are Qwen, created by Alibaba Cloud..."). Always include an explicit system prompt for Qwen 3.5.

### Failure surface (verified, Qwen 3.5 2B)

| Failure class | Qwen 3.5 2B |
|---|---|
| Implicit negation | low |
| Adjective filter | low |
| **Substitution bias** | **high** — characteristic failure |
| Markdown emission | low |
| Thinking-on impact | **−44pp** native thinking → `<think>` leak in content |
| Multi-pass | neutral (no gain, no loss) |

### Recommendations

**For task-facing prompts on Qwen family:**
1. Use the skeleton + **CRITICAL: rules are sacred** anti-substitution block at top (see `techniques-small.md § Anti-pattern named block`)
2. **5 examples at END** (one more than Gemma — Qwen needs the extra example for the substitution-prevention case)
3. **Always pass `enable_thinking=False`** for compile-time tasks. Don't rely on soft switches alone.
4. For thinking-enabled tasks: recommended params are T=0.6, TopP=0.95, TopK=20, MinP=0 ([source](https://huggingface.co/Qwen/Qwen3-8B)) — but again, default thinking-off for 2-3B compile tasks

### Variant: Qwen 3.5 9B
- Better implicit-negation handling than 2B
- Substitution bias drops to "medium"
- Thinking-on still hurts but less drastically (~−15-25pp)
- Worth trying when 2B hits ceiling but you don't want frontier-cost

---

## Microsoft Phi family

### Lineup (2026)
- **Phi-4-mini 3.8B** — instruct variant
- **Phi-4-mini-reasoning 3.8B** — separate variant with thinking enabled
- **Phi-4 14B** — full size, beyond "small" but related
- **Phi-4 multimodal** — image+audio+text input

### License
- MIT license — cleanest of all in this category for commercial use

### The defining quirks (per Microsoft model card)

1. **Function-calling hallucinations**: "with function calling scenarios, the model could sometimes hallucinate function names or URL's" — direct quote from model card
2. **Long-conversation drift**: "can in some cases generate responses that are repetitive, unhelpful, or inconsistent in very long chat sessions" — limit turns
3. **Multilingual gaps**: "languages other than English will experience worse performance" — same-language attack prompts can cause unsafe output
4. **Code limited to Python with common packages**: typing, math, random, collections, datetime, itertools. For other packages/languages, verify all API uses manually.

### Chat format

Phi-4 uses a specific chat template with control tokens:
```
<|system|>Insert System Message<|end|><|user|>Insert User Message<|end|><|assistant|>
```

For function calling:
```
<|system|>You are a helpful assistant with some tools.<|tool|>[{...tool JSON schema...}]<|/tool|><|end|><|user|>...<|end|><|assistant|>
```

Tools must be wrapped in `<|tool|>...<|/tool|>` tokens (not standard OpenAI `tools` parameter format).

### Failure surface (not verified in our suites — based on vendor docs and community reports)

| Failure class | Phi-4-mini 3.8B |
|---|---|
| Implicit negation | medium (not measured in our suites) |
| Adjective filter | medium |
| Substitution bias | low-medium |
| Markdown emission | low |
| Thinking-on impact | n/a (separate `reasoning` variant exists) |
| Function calling hallucinations | **high** (vendor documented) |
| Long-conversation drift | high |

### Recommendations

**For task-facing prompts on Phi-4-mini:**
1. Use the skeleton with standard structure
2. **Avoid function calling on Phi-4-mini-instruct** without 3-5 few-shot tool-call examples — vendor admits hallucinations
3. For RAG / long-context tasks: keep conversation turns minimal, restart history when possible
4. Don't deploy Phi-4-mini for non-EN tasks without explicit testing — vendor warns of multilingual gaps
5. For reasoning-heavy compile tasks: try `Phi-4-mini-reasoning` variant instead of forcing thinking on the instruct variant

### Variant: Phi-4-mini-reasoning
- Native thinking enabled, like Qwen 3 reasoning mode
- Better suited for multi-step planning tasks
- Same chat template + thinking blocks
- Apply standard thinking-mode anti-patterns from Qwen section if forcing thinking off

---

## Meta Llama family

### Lineup (2026, small/medium)
- **Llama 3.2 1B / 3B** — small text-only
- **Llama 3.3 70B** — outside small category
- **Llama 3.2 11B / 90B vision** — multimodal
- **Notable fine-tunes**: Hermes-3-Llama-3.2-3B (Nous Research), SmolLM family (HF, not Llama-based but adjacent)

### License
- Llama 3 Community License — permissive for inference but restrictive on training large derivatives (>700M MAU clause)

### Chat format

Llama 3 / 3.2 uses standard chat template:
```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>
{system_prompt}<|eot_id|>
<|start_header_id|>user<|end_header_id|>
{user_message}<|eot_id|>
<|start_header_id|>assistant<|end_header_id|>
```

System role supported — no Gemma-style quirk.

### Failure surface (not extensively verified in our suites)

Llama 3.2 3B is positioned by Meta for "assistant-like chat and agentic applications" — not for compile-time NL→DSL. Limited in-house data, but consistent with the broader 3B pattern:

| Failure class | Llama 3.2 3B |
|---|---|
| Implicit negation | medium-low |
| Adjective filter | medium |
| Substitution bias | low-medium |
| Markdown emission | low-medium |
| Thinking-on impact | n/a (no native thinking) |
| Tool calling | JSON-based template recommended by Meta |

### Recommendations

**For task-facing prompts on Llama 3.2 3B:**
1. Use the skeleton with system role (Llama supports it cleanly)
2. JSON tool calling template per Meta's recommendation when tools are needed
3. Test on a small benchmark — Llama 3.2 3B is less profiled than Gemma/Qwen/Mistral in our suites, so the matrix-small recommendations have lower confidence

### Variant: Hermes-3-Llama-3.2-3B (Nous Research)
- Function-calling-tuned Llama 3.2 3B
- Strong on tool-call adherence — one of the better small-model picks for native function calling
- General instruct quality slightly behind base Llama 3.2 3B for non-tool tasks
- License inherits Llama 3 Community License

---

## Russian-tuned variants (when input is RU and target is small local)

The EN-system unlock (`antipatterns-small.md § 6`) is the right default for base 2-3B models on RU input. **But** dedicated RU tunes invert this — same-language system can win.

### saiga_gemma3_12b (IlyaGusev)
- Heavy Russian corpus shift on Gemma 3 12B base
- RU system + RU input often beats EN system + RU input
- Best small-model pick for RU narrative / creative / roleplay tasks
- Inherits Gemma's "no system role" quirk → fold system into first user turn

### saiga_gemma4 (not released as of 2026-05)
- IlyaGusev has saiga line on multiple bases — gemma3 is current. Gemma 4 saiga had not appeared as of last check.

### T-lite-it-2.1 (Yandex)
- 7B Russian instruct, strong on RU intent classification and dialog
- Apache 2.0 (Yandex released openly)
- Better RU coverage than base Qwen / Mistral / Gemma at the same size, but English performance behind

### RuGPT family (Sber)
- Older generation, less competitive on instruction-following than saiga / T-lite for new builds
- Still in some production deployments — note if user mentions

### Recommendation when target is RU + small + local
1. Check if dedicated RU tune exists for the base (saiga for Gemma, T-lite as standalone)
2. If yes: use RU system prompt (same language as task input)
3. If no (using base Gemma / Mistral / Qwen / Phi / Llama): apply EN-system unlock from `antipatterns-small.md § 6`

---

## When the user names a model not listed here

Match by name pattern (most discriminating prefix first):

| Name pattern | Family to use |
|---|---|
| `phi-*` | Phi family |
| `gemma-3-*` or `gemma3-*` | Gemma 3 |
| `gemma-4-*` or `gemma4-*` or `gemma-*-e[24]b` | Gemma 4 |
| `qwen-3.5-*` or `qwen3.5-*` | Qwen 3.5 |
| `qwen-3-*` or `qwen3-*` (not 3.5) | Qwen 3 |
| `qwen-2.5-*` | Qwen 2.5 (treat as Qwen 3 family but expect higher substitution bias) |
| `ministral-*` or `mistral-*-3b` or `mistral-*-7b` | Mistral / Ministral |
| `llama-3.2-*` or `llama3.2-*` | Llama 3.2 |
| `hermes-3-*` | Hermes (Llama-based) |
| `saiga_*` | saiga (Russian Gemma tune) |
| `t-lite-*` or `tlite-*` | T-lite (Russian Yandex) |
| `*-instruct` | match base model family, then add instruct expectations |
| `*-reasoning` or `*-thinking` | family + native thinking enabled (apply Qwen thinking-mode anti-patterns) |

**Flag explicitly to the user which family you used as proxy** so they can override if direct experience differs.
