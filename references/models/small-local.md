# Small local model families — vendor deltas

Companion to [matrix-small.md](../matrix-small.md). Family-level notes that don't fit the matrix cells: known patho failures, fine-tune variants, vendor-specific quirks, and recommended starting prompt for each family.

When the user's model is listed here — read its section. When not listed — match to the closest family by name pattern (see `matrix-small.md § When the matrix doesn't cover the model`) and read that family's notes.

This file is about **prompt wording**. Sampling values, template kwargs and serving flags appear only where they change what the prompt has to say or what a failure gets blamed on. No prices, no context-size comparisons.

**Class 2 boundary.** Class 2 = models run locally for single-pass task prompts: **dense 2–9B**, or **MoE with ≤ ~9B active AND ≤ ~35B total parameters** (fits one consumer GPU or a 32–64 GB Mac). Anything above that is Class 1 (frontier files). By this boundary some MoE models of families already covered here now count as Class 2 without having rows: Gemma 4 26B MoE and Qwen3.6-35B-A3B (both noted in `FINDINGS.md § П8`, 2026-08-29). Use the family's row as a proxy and say so. Dense 12B+ models listed below (Gemma 3 12B/27B, Mistral Nemo 12B, Phi-4 14B, saiga_gemma3_12b) are above the boundary. They stay only as family context.

**Not covered: decision models.** Cloudflare Clef / Clef-flash, Amazon Strands Decider, inclusionAI LemonDecider, Perplexity pplx-decider, LiquidAI d1 and the `decision` adapter of IBM Granite Switch take a state plus typed questions and return calibrated probabilities over option labels in one forward pass, with no system prompt (`FINDINGS.md § П10`, 2026-10-14). They are not prompted with task prose, so this file has no rows for them. What you can word there is the vendor's `instructions` / `criteria` text, and the vendor's own invocation recipe beats any rewrite. That is outside the atlas.

---

## Google Gemma family

### Lineup (2026)
- **Gemma 3 1B / 4B / 12B / 27B** — text-only, dense decoder. 128k context (4B+).
- **Gemma 4 e2b / e4b** — "edge" variants, designed for on-device. Smaller param count, larger activations than 2B/4B equivalents (e2b ≈ 2B params + extra capacity through E-mixing).
- **Gemma 3n / 4n** — multimodal variants (image+text input).
- **Notable fine-tunes**: HORROR-Imatrix (gemma-3-4b, horror-tuned), saiga_gemma3_12b (Russian instruct tune), TrevorJS uncensored variants.
- **No newer Gemma generation as of 2026-10-31.** The `google` HF org's newest repo is `embeddinggemma-2` (2026-09-14). E2B-it / E4B-it are unchanged since 2026-07-20. `gemma-4-12B-it` (above the boundary) got a tokenizer-only patch on 2026-10-30 that fixes 210 glitch tokens, with "No model weights are modified". If you run that build, re-pull `tokenizer.json` before you blame odd output tokens on the prompt.

### License
- **Gemma 4: Apache 2.0** (Google switched from custom Gemma license for v4). Clean for commercial deployment.
- Gemma 3 and earlier: custom Gemma Terms of Use (restrictive on redistribution but permissive for inference).

### ⚠️ Weight re-pull note — Gemma 4 stealth update (2026-07-16)

Google re-published the Gemma 4 weights **under the same name** (no version bump, no "4.1") on 2026-07-16: it fixes broken tool calling and prematurely truncated / incomplete responses, and adds Flash Attention 4 (FA4) prefill acceleration on Nvidia Hopper GPUs (prompt-processing +25–70%, time-to-first-token −31% — Hopper H100+ only; Ada/RTX 40-series excluded from the FA4 gains, though they get the fixes). **If the local harness is running any Gemma 4 weights pulled before 2026-07-16, re-pull from Hugging Face** — the improvements live in the weights, not in config. This matters for prompt review because a harness sitting on the broken weights will misattribute tool-call failures or truncation to the prompt. Record the weight download date alongside any benchmark result as a de facto version identifier.

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
- **Ministral 3 (2512)**: the current small Mistral generation. Sizes 3B / 8B / 14B, each as Base, Instruct and Reasoning, Apache 2.0. 3B and 8B are Class 2. 14B is dense and above the boundary (`mistral-frontier.md`).
- **Ministral 3B / 8B (2024)**: the API ids `ministral-3b-2410` / `-8b-2410` were retired on 2025-12-31, and Mistral names Ministral 3 3B / 8B as the replacement. **The measured "Ministral 3B" rows in `matrix-small.md` do not record which build they ran on.** On a Ministral 3 (2512) build, treat those cells as a proxy until re-measured.
- **Mistral 7B v0.3**: original instruct, still useful baseline
- **Mixtral 8x7B / 8x22B**: MoE, above the Class 2 total-parameter bound. Listed only as the family pattern.
- **Mistral Nemo 12B** (Mistral + NVIDIA collab): dense 12B, above the boundary. Its API id was retired on 2026-07-31 in favour of Ministral 3 8B.
- **Codestral**: code-tuned variants

### Vendor guidance on the Ministral 3 card (verbatim)
- "**System Prompt**: Maintains strong adherence and support for system prompts." and "System Prompt: Define a clear environment and use case, including guidance on how to effectively leverage tools in agentic systems."
- "Use a **temperature below 0.1** for daily-driver and production environments". This matches the `temperature=0.1` default in `antipatterns-small.md § 11`.
- "Keep the set of tools well-defined and limit their number to the minimum required for the use case". This is the same direction as the small-model tool rule (`antipatterns-small.md § 4`).

### License
- **Ministral 3 (2512): Apache 2.0**, clean for commercial use
- Caveat: an earlier Mistral Research License applied to some 2024 8B variants, so verify the exact build before commercial deployment
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

### Mistral-adjacent guard model: Shieldstral (3B, 2026-08-04)

Mistral's Shieldstral (`mistralai/Shieldstral-1.0-3B`, **3B, open weights under Apache 2.0**) is a **multimodal safety classifier**, not a generative model: it frames moderation as a policy-adaptive Q&A task (you write the policy as a plain-language question at inference; it returns a calibrated safety score over a single token — no retraining, one interface for text and images). Vendor reports it matches open guard models **up to 7× its size** on text-safety / refusal benchmarks and sets SOTA on multimodal moderation; runs on a single 16GB GPU. Niche for prompt work: **guard-model for task pipelines**, not a prompt target itself — noted here so a later coverage pass doesn't re-investigate it.

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
- **No new small Qwen as of 2026-10-31.** The `Qwen` HF org's newest repo is `Qwen-Image-2.1-Turbo` (2026-10-09), and nothing was modified after 2026-10-14. Qwen3.6-35B-A3B (3B active) falls inside the Class 2 MoE boundary but has no row. Use the Qwen 3.5 9B row as a proxy and flag it.
- **Derived 9B: `XiaomiMiMo/MiMo-V2.6-Distill-Qwen-9B`.** `base_model: Qwen/Qwen3.5-9B`, MIT, SFT only. The card describes it as "a starting point for open research in agentic reinforcement learning". Thinking is switched through `chat_template_kwargs.enable_thinking`, and the card gives no sampling recommendation. Treat it as a Qwen 3.5 9B tune: same `enable_thinking=False` rule, axes `?`.

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

## IBM Granite family (added 2026-08-29)

### Lineup (2026)

- **Granite 4.2 3B / 8B** — in the atlas's 2-9B class. Dense, decoder-only **reasoning** models, Apache 2.0, weights on Hugging Face (`ibm-granite/granite-4.2-3b` / `-8b`; also `-30b`)
- **Granite 4.2 30B** — above the 2-9B range (like Gemma 4 31B / Qwen3.6-27B, noted but not covered by matrix-small)
- First Granite generation built around **explicit reasoning**; earlier Granite 4.0/4.1 were instruction-following assistants
- **Granite 4.2 is still the newest Granite generation found.** An HF name search for `granite-4.3` on 2026-10-31 returned only third-party repos.
- **Granite Switch 4.1 8B** (`ibm-granite/granite-switch-4.1-8b`; revision 4.1 on 2026-10-14, 4.1.1 on 2026-10-27; also 30B and 3B/8B/30B `-preview`). This is not a new generation. It is "a modular LLM that composes a base model, multiple adapters and an encoder": a `granite-4.1-8b` base plus the `decision` and `guardian-core` aLoRA adapters plus an embedded 470M speech encoder. "Adapters are activated on demand via control tokens in the chat template". You pick one per request with `chat_template_kwargs: {"adapter_name": …}`, not in prose. Chat answers come from the **4.1** base, so the 4.2 thinking-mode notes below do not apply. The `decision` adapter is a decision model and is not covered (see the top of this file).

### License

- **Apache 2.0** (weights, quantized variants, docs) — clean for commercial deployment

### The defining quirk: switchable thinking in the chat template

Each Granite 4.2 model exposes **three operating modes through its chat template**:

1. **Thinking** (default) — full chain of thought inside dedicated tags before the answer
2. **Non-thinking** — answers directly
3. **Low-effort** — short reasoning budget for easy questions

**Mode is a template/serving choice, not prompt prose.** A compile-time task prompt should not contain "don't think" — pick the non-thinking / low-effort mode in the harness call (same class of rule as every frontier vendor's out-of-band reasoning knob; `antipatterns-small.md` thinking-on regression applies by default if you leave thinking on for 2-3B compile tasks).

**Multi-turn gotcha:** in multi-turn conversations, previous turns' thinking is **stripped by default** (template behavior) — never write prompts that depend on visible CoT from earlier turns, and don't expect `reasoning_content`-style continuity.

### Native tool calling

The same template reasons about which tool to call and why before emitting the call, in **OpenAI function-calling format** — plugs into vLLM / SGLang harnesses without extra adapters. The small-model rule still applies: below ~7B, tool calls need 3-5 few-shot tool-call examples (`antipatterns-small.md § 4`).

### Training context that affects prompting

- Pre-trained on ~15T tokens, context window grown to **512K** via a five-phase schedule (the longest among the Class 2 models profiled here; usable-window behavior at scale unverified)
- SFT on ~7.2M samples (CoT, reasoning, agentic trajectories)
- **8B (and 30B): multi-stage RL inside live environments** (OpenHands repo edits graded by hidden tests; terminal with up to 64 turns; multi-hop web search) — the first Class 2 models in the atlas with environment-RL training, and the reason agentic-loop prompts are viable on the 8B where other small models regress (still: scope tightly, outcome-defined). Since 2026-10, Mellum2.1-12B-A2.5B (RL in real repositories) and LFM2.5-24B-A2B (agentic RL inside agent harnesses) also report it, with axes `?`
- RLHF adds a **reasoning-length penalty** — verbose CoT chains are discouraged by training
- 3B skips the agentic-RL block — treat it as a plain small reasoning model
- 12 languages tested (EN, DE, JA, AR, KO, ZH, …) — non-EN coverage exists, per-model behavior unmeasured in our suites

### Benchmarks (vendor-reported)

| | 3B | 8B |
|---|---|---|
| SWE-Bench Verified | — | 47.67 |
| Terminal-Bench 2.1 | — | 20.56 |
| AIME25 | 78.33 | 86.67 |
| GPQA | 54.80 | 64.14 |
| RULER @128K | 55.30 | 71.41 |

### Recommendations

**For task-facing prompts on Granite 4.2 3B/8B:**
1. Use the standard skeleton (`techniques-small.md`) — system role is supported, no Gemma-style fold-in
2. **Pick the thinking mode at serving time** — non-thinking / low-effort for compile-time tasks; thinking default only where reasoning depth is the point
3. Native tool calls OK, but add few-shot tool-call examples on the 3B; the 8B tolerates agentic loops better (environment-trained) — still define success criteria and stop conditions
4. Failure-surface cells are **`?`-untested in our suites** — don't extrapolate from Gemma/Qwen numbers; run a small suite before committing

### Quantized variants

FP8 (no calibration), NVFP4 / MXFP4 (calibrated), fourteen GGUF formats via llama.cpp — same quant-choice testing advice as other families (Q4 vs Q8 is task-dependent).

---

## OpenBMB MiniCPM5 (added 2026-10-31)

`openbmb/MiniCPM5-2B` (HF repo created 2026-09-06): **2.52B dense** ("Standard `LlamaForCausalLM`"), Apache 2.0, ChatML, system role supported. All behavioral axes are `?` because it has not been run on our suites. The four facts below change what a review checks.

1. **`enable_thinking` has three states, not two.** In the template, `False` → an empty `<think>\n\n</think>\n\n` block, `True` → an open `<think>\n`, and **absent → nothing appended, so the model decides**. Leaving the kwarg out does not mean thinking is off. For compile-time tasks pass `False` explicitly. The multi-provider block in `techniques-small.md § Triggering thinking-off` already does this.
2. **The `min_p` trap is a harness default, not a prompt failure.** The card recommends "`temperature=1.0, top_p=0.95, min_p=0.0`" and warns: "In llama.cpp, the default `min_p=0.05` can lead to repetitive output: it filters out tokens whose probability is below 5% of the highest-probability token, potentially discarding the exact tokens needed to break out of a repetition loop." If the output loops on llama.cpp, check `min_p` before you rewrite the prompt. For persistent repeats the card adds `repetition_penalty=1.05`.
3. **`--jinja` is required on `llama-server`** (the card's launch line). Without it the template, including the thinking switch above, is not applied.
4. **Tool calls are XML, not JSON.** "MiniCPM5-2B emits XML-style tool calls", in the shape `<function name="…"><param name="…">value</param></function>`. Values containing `<`, `&` or newlines go in `<![CDATA[…]]>`. The template appends its own tool-usage guidelines after the system prompt, or substitutes them where the system prompt contains the literal marker `<tool_def_sep>`, so you choose where the tool block lands. Tool results come back as a `user` turn wrapped in `<tool_response>`. If you add few-shot tool-call examples (small-model rule, `antipatterns-small.md § 4`), write them in this XML shape. An OpenAI-JSON example contradicts the format the template instructs. SGLang (`--tool-call-parser minicpm5`) is the vendor's recommended tool backend.

---

## NVIDIA Nemotron-Labs Dory (added 2026-10-31)

`nvidia/NVIDIA-Nemotron-Labs-Dory1.0-4B-BF16` (also MXFP8 / NVFP4 / GGUF), released 2026-10-16: **4B looped (latent-recurrent) model**, English only, OpenMDW 1.1, "General-purpose reasoning on a small footprint". Behavioral axes are `?`.

- **Depth is a loop count, not a thinking budget.** A shared block of layers runs `n_recurrent_loops` times (vLLM `--hf-overrides`; llama.cpp `dory.num_loops` = "`1`, `2`, `3`, or `4`"). The card: "We recommend **two loops** as the default starting point". It suggests "**four loops** when additional accuracy matters more, particularly for agentic tasks". This is serving config. No prompt wording changes the loop count, so a review that wants more depth names the setting and leaves the prompt alone.
- **Reasoning is on by default.** Card: "Reasoning is always on in vLLM and enabled with `--reasoning on` in the llama.cpp command". So the small-model thinking-off default (`antipatterns-small.md § 2`) is not a documented option here. Budget `max_tokens` for the trace and strip it (`techniques-small.md`). The shipped `chat_template.jinja` does accept three **undocumented** kwargs, all untested by us: `enable_thinking` (default True; False renders `<think></think>`), `medium_effort` (appends `\n\n{reasoning effort: efficient}` to the last user message, so if you see that string in a prompt dump, it came from the kwarg) and `truncate_history_thinking` (default True; drops reasoning from turns before the last user message).
- **`force_nonempty_content` is not in the shipped template.** The card says: "For coding agents, add `extra_body={"chat_template_kwargs": {"force_nonempty_content": True}}`". The template, unchanged since the 2026-10-16 upload, has no reference to it, and neither does `tokenizer_config.json`. As shipped it is a no-op unless the serving code handles it (the vLLM PR was not read). The card does not say what the flag guards against, though the name points at empty `content`. If coding-agent turns come back with empty content, look at the harness. Neither the flag nor prompt prose is a documented fix.
- **Tool format** is `<tool_call><function=name><parameter=p>…</parameter></function></tool_call>` (vLLM parser `qwen3_coder`, reasoning parser `nemotron_v3`). With tools present, the template appends an `<IMPORTANT>` "Reminder" block on that format to the system prompt, so a hand-written copy of the same rules only duplicates it.
- **Sampling:** "Temperature 1.0" and "Top_P 0.95"; the llama.cpp line uses `--top-k 0 --min-p 0 --repeat-penalty 1`. The project default of `temperature=0.1` (`antipatterns-small.md § 11`) is not what the vendor tested.
- **English only**, in both pre-training and post-training. Non-EN task input is `?`. Keep the system prompt in English.

---

## LiquidAI LFM (added 2026-10-31)

Liquid's blog "Introducing LFM3" (2026-10-27): "The first release of the LFM3 family covers models in the 2B parameter range." License: LFM Open License v1.0. All behavioral axes are `?`.

### LFM3-1.7B: text model of the new generation

`LiquidAI/LFM3-1.7B`: **1.73B**, just under the 2B lower bound. It is covered here as the family's text model; flag that when you use it as a target. It is a hybrid of short convolutions and attention, with 16 languages including Russian. The card positions it for "local agents, tool calling, structured outputs, and domain-specific fine-tuning". Also: "Use retrieval or search for knowledge-heavy tasks. It is not recommended for agentic coding or complex multi-step reasoning due to its small size."

- **Thinking off is not reliable.** "Reasoning is enabled by default. You can set `enable_thinking=False` to disable thinking traces, but this behavior can be inconsistent, especially for reasoning-heavy tasks." So pass `False`, and also keep the strip-and-budget safety net (`techniques-small.md § Triggering thinking-off`). "Reasoning from turns before the latest user message is removed unless `preserve_thinking=True`." That is the same multi-turn rule as Granite 4.2.
- **Sampling by task class:** "**General** (chat, tool use, agents, coding): `temperature=0.1,top_k=50, repetition_penalty=1.1`"; "**Math & STEM**: `temperature=1.0, top_p=0.95, top_k=20`".
- **Template:** ChatML. The card's format example carries the default system line "You are a helpful assistant trained by Liquid AI.", so send your own system prompt. Tool calls go in `<tool_call>` XML, and `function.arguments` "must be a Python mapping" (harness).
- **Deferred tools have a vendor system-prompt line.** With `defer_loading=True` the tool definitions stay out of the prompt until the harness's search tool returns them. The card's wording for the system prompt (verbatim): "You only have one tool right now: search_tools, which searches a catalog of many more tools. If none of your current tools can handle a request, call search_tools to find one, then call the tool it returns."
- **Siblings:** `LFM3-Omni-2B` (2.17B) is "a **non-reasoning** model". "By default, the model selects the output modality based on the system prompt and input modality, typically matching the input modality. To encourage a specific output modality, append the corresponding instructions to the end of your system prompt." For text-only output the card's line is `Respond with text.` (with `generation_mode="text"`). ASR is requested with the system line "Perform ASR in <language>.". `LFM3-VL-2B` (1.92B, VLM) is "not recommended for coding, multi-turn conversations, or complex reasoning". `d1-2B` / `d1-Omni-2B` are decision models (not covered, see the top of this file).

### LFM2.5-24B-A2B: MoE inside the boundary, thinking-only

`LiquidAI/LFM2.5-24B-A2B` (HF 2026-10-12): "Total parameters: 24.0B / Active parameters: 2.45B". That puts it in Class 2 by the MoE rule. 16 languages including Russian; vendor sampling `temperature: 0.1` / `top_k: 50` / `repetition_penalty: 1.05`.

- **No thinking-off switch.** "LFM2.5-24B-A2B is a pure reasoning model that always thinks before it answers. It adds a `<think>` tag directly in the chat template". The thinking-off default in `antipatterns-small.md § 2` cannot be applied. Budget `max_tokens` for the trace, strip it, and don't write "don't think" in the prompt, because no switch exists for prose to stand in for.
- **Tool-call format is a prompt choice.** "By default, LFM2.5 writes Pythonic function calls (a Python list between `<|tool_call_start|>` and `<|tool_call_end|>` special tokens) … You can override this behavior by asking the model to output JSON function calls in the system prompt." This is one of the few Class 2 models where the vendor documents a format switch made in prose.
- **Harness-native.** Agentic RL ran "inside popular agent harnesses (OpenClaw, Hermes Agent, Pi, OpenCode), so the model learns their tools, system prompts, and interaction patterns." Inside those harnesses, their stock system prompt is the trained-on baseline. Compare against it before you rewrite.
- "It is not recommended for knowledge-heavy tasks on its own: pair it with retrieval or web search".

---

## Noted, not profiled

- **JetBrains Mellum2.1-12B-A2.5B-Thinking**: 12B total / 2.5B active (inside the MoE boundary), Apache 2.0, `language: en` only. The card calls it "a thinking model" (no off-switch documented) for code, repositories and tool use (`--reasoning-parser qwen3`, `--tool-call-parser hermes`; card example temperature 0.6, top_p 0.95, top_k 20). It is too narrow for a general Class 2 row. If it is the target, apply the no-off-switch notes from LFM2.5-24B-A2B and keep prompts in English.
- **Specialised translation models, not general Class 2:** `IndexTeam/Index-Translate-9B` (also 2B and 35B-A3B-preview; "built on Qwen3.5", 150 languages) ships its own Chinese prompt template with hard and soft constraints (`【约束要求】` with `【硬性要求】` / `【注意】`), greedy `temperature=0` and thinking disabled. `yandex/Yandex-8B-WMT26` (~8B, en → ru / be / kk / hy only) is called with no chat template, through a raw instruction such as `Переведи с английского на казахский:` followed by the text, with greedy decoding (`do_sample=False`) and a domain value (`social` / `speech` / `news` / `software`). For both, use the vendor's template verbatim. The task-prompt skeleton in this atlas does not apply.

---

## Russian-tuned variants (when input is RU and target is small local)

The EN-system unlock (`antipatterns-small.md § 6`) is the right default for base 2-3B models on RU input. **But** dedicated RU tunes invert this — same-language system can win.

### saiga_gemma3_12b (IlyaGusev)
- 12B dense, so it is **above the Class 2 boundary**. Kept here because it is the only Gemma-based RU tune in this file. No ≤9B saiga Gemma exists (next entry).
- Heavy Russian corpus shift on Gemma 3 12B base
- RU system + RU input often beats EN system + RU input
- Best small-model pick for RU narrative / creative / roleplay tasks
- Inherits Gemma's "no system role" quirk → fold system into first user turn

### saiga_gemma4 (not released as of 2026-10-31)
- IlyaGusev has saiga line on multiple bases — gemma3 is current. No Gemma 4 saiga: the `IlyaGusev` HF org's newest repo dates from 2026-01 (scan 2026-10-31).

### T-lite-it-2.1 (T-Bank, HF org `t-tech`, not Yandex)
- 7B Russian instruct, strong on RU intent classification and dialog
- License: check the `t-tech` model card before deployment (not re-read here). The org's newest repo is T-Search (2026-07-29), and no newer T-lite appeared as of 2026-10-31.
- Yandex's own new small model in the window is `Yandex-8B-WMT26`, a translation-only model (§ Noted, not profiled).
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
| `ministral-3-*-2512` or `ministral-3-*` | Ministral 3 (2512); measured Ministral 3B cells as proxy, flag it |
| `ministral-*` or `mistral-*-3b` or `mistral-*-7b` | Mistral / Ministral |
| `minicpm5-*` or `minicpm-*` | MiniCPM5 (tri-state `enable_thinking`, `min_p=0`, XML tools) |
| `*dory*` or `nemotron-labs-*` | Nemotron-Labs Dory (loops, reasoning on) |
| `lfm3-*` or `lfm2.5-*` or `lfm-*` | LiquidAI LFM (check which: LFM3 text / Omni / VL, or LFM2.5 MoE) |
| `granite-switch-*` | Granite Switch (4.1 base + adapters); `decision` adapter not covered |
| `*-distill-qwen-*` | base Qwen family of the named size (e.g. MiMo-V2.6-Distill-Qwen-9B → Qwen 3.5 9B) |
| `mellum*` | Mellum (§ Noted, not profiled) |
| `*-translate-*` or `*-wmt*` | specialised translation model: vendor template verbatim, not the task skeleton |
| `*decider*`, `clef*`, `d1-*` | decision model, not covered (top of file) |
| `llama-3.2-*` or `llama3.2-*` | Llama 3.2 |
| `hermes-3-*` | Hermes (Llama-based) |
| `saiga_*` | saiga (Russian Gemma tune) |
| `t-lite-*` or `tlite-*` | T-lite (Russian, T-Bank) |
| `*-instruct` | match base model family, then add instruct expectations |
| `*-reasoning` or `*-thinking` | family + native thinking enabled (apply Qwen thinking-mode anti-patterns) |

**Flag explicitly to the user which family you used as proxy** so they can override if direct experience differs.

---

## Source notes (2026-10-31 additions)

All read 2026-10-31 (raw). These are HF `raw/main/README.md` files, chat templates or HF API JSON, unless tagged otherwise.

- MiniCPM5-2B: https://huggingface.co/openbmb/MiniCPM5-2B (README + `chat_template.jinja`). Covers parameter count, sampling and the `min_p` warning, `--jinja`, the tri-state `enable_thinking`, the XML/CDATA tool format, `<tool_def_sep>` and `<tool_response>`.
- Nemotron-Labs Dory: https://huggingface.co/nvidia/NVIDIA-Nemotron-Labs-Dory1.0-4B-BF16 (README, `config.json`, `chat_template.jinja`, commit list). Covers loops, "Reasoning is always on in vLLM", sampling, parsers and `force_nonempty_content`. The 0 hits for that flag in the template and tokenizer config came from a grep of the shipped files. vLLM PR #60149 was **not read**.
- LFM3: https://www.liquid.ai/blog/lfm3 (2026-10-27); https://huggingface.co/LiquidAI/LFM3-1.7B; https://huggingface.co/LiquidAI/LFM3-Omni-2B; https://huggingface.co/LiquidAI/LFM3-VL-2B; https://huggingface.co/LiquidAI/d1-2B.
- LFM2.5-24B-A2B: https://huggingface.co/LiquidAI/LFM2.5-24B-A2B.
- Mellum2.1: https://huggingface.co/JetBrains/Mellum2.1-12B-A2.5B-Thinking.
- Granite Switch 4.1: https://huggingface.co/ibm-granite/granite-switch-4.1-8b. "No Granite 4.3" rests on an HF `search=granite-4.3` query only, so it is not a full org scan.
- Index-Translate-9B: https://huggingface.co/IndexTeam/Index-Translate-9B. Yandex-8B-WMT26: https://huggingface.co/yandex/Yandex-8B-WMT26.
- MiMo-V2.6-Distill-Qwen-9B: https://huggingface.co/XiaomiMiMo/MiMo-V2.6-Distill-Qwen-9B (raw README, via the china research pass of 2026-10-31).
- Ministral 3 (2512): https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512 (card guidance quoted above) and https://docs.mistral.ai/models.md ("Deprecated and retired models": `ministral-3b-2410` / `-8b-2410` retired 2025-12-31, `open-mistral-nemo-2407` retired 2026-07-31).
- Gemma 4 12B tokenizer patch: HF API commits for `google/gemma-4-12B-it` plus discussion #55. "No new Gemma / small Qwen / Phi / SmolLM / Llama" rests on HF API scans of the `google`, `Qwen`, `microsoft`, `HuggingFaceTB` and `meta-llama` orgs, sorted by createdAt and lastModified. That method misses repos created earlier and made public later, and it misses new orgs.
- Russian orgs (`t-tech`, `ai-sage`, `Vikhrmodels`, `IlyaGusev`, `yandex`): HF API scans by the same method.
- Class 2 boundary and the decision-model exclusion: owner decision recorded for v2.0.0 (2026-10-31). Gemma 4 26B MoE and Qwen3.6-35B-A3B come from `FINDINGS.md § П8` (2026-08-29), not re-read today.

