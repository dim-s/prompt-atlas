# Anti-patterns for small local models

Companion to [matrix-small.md](matrix-small.md) and [techniques-small.md](techniques-small.md).

These are concrete techniques that work on frontier models (Claude, GPT-5.x, Gemini 3) but **regress small local models (2-9B)** when applied. Every anti-pattern listed has been reproduced on our `nl_dsl_game` / `nl_rules_farm` suites or has documented vendor / research backing.

When reviewing a small-model prompt, scan for these. Each is a `[CRITICAL]` or `[IMPROVE]` finding when found.

---

## 1. Abstract principles mid-prompt instead of examples

**The anti-pattern:**
```
## Strict reading of rule conditions

Every adjective in a rule condition is mandatory.
Check negations carefully.
Pay attention to "but not", "except", and similar exception markers.
The rules are sacred — do not paraphrase them.
```

Placed in the middle of the prompt body, between the whitelist and Forbidden.

**Why it regresses small models:**
- Abstract principles like "be careful" / "pay attention" / "every X is mandatory" have no concrete training-data analog the model can pattern-match
- Placement in the middle of a long prompt loses attention even when content is correct (`matrix-small.md § Critical rules position`)
- Multiple principles in series cascade into noise — model ignores all of them

**Reproduced regression** (`nl_rules_farm` suite, 32 cases):
- Gemma 4 e2b: 93% → 86% (−7pp)
- Ministral 3B: 79% → 64% (−15pp)
- Qwen 3.5 2B: 64% → 64% (flat, but the time spent was wasted)

**What to do instead:**
Replace principles with **demonstrations**. For each principle you wanted to state, add a few-shot example showing the success path on that exact failure mode. Place examples at END, not middle. See `techniques-small.md § Few-shot example template`.

If you must keep some text instruction, use **named anti-patterns** (`techniques-small.md § Anti-pattern named block`) — "Never substitute X for Y" with a concrete example beats "be careful with substitutions".

**Backing:** Google's Gemma docs warn that "Gemma 3+ loses negative constraints in the middle of the prompt." Research consensus: ["LLMs exhibit rule-rigidity; simple prompting techniques failed to correct."](https://arxiv.org/abs/2503.22395)

---

## 2. Thinking-mode on for 2-3B models

**The anti-pattern:**
```python
# Calling Qwen 3 / Gemma 4 / Phi-4-mini-reasoning with thinking enabled
extra_body = {
    "chat_template_kwargs": {"enable_thinking": True},
    "reasoning_effort": "high",
}
# Or simply not passing the off-flags, and using a model with thinking-on default
```

**Why it regresses small models:**
- Thinking blocks (`<think>...</think>`) eat the `max_tokens` budget; structured output gets truncated → `unparseable`
- Without provider-specific cleanup (`--jinja` for llama-server, etc.), thinking tokens leak into `content` and contaminate JSON parsers
- 2-3B models don't have the capacity for long CoT to actually improve answer quality on most tasks — they spin and degrade
- Research consensus: ["Small student models ≤3B parameters do not consistently benefit from long CoT reasoning"](https://arxiv.org/pdf/2502.12143)

**Reproduced regression** (`nl_dsl_game` suite):
- Gemma 4 e4b: 94% → 50% (−44pp, all failures `unparseable`)
- Gemma 4 e2b: 75% → 50% (−25pp)
- Qwen 3.5 2B: 75% → 31% (−44pp)

**What to do instead:**
Default to `thinking=off` across all small-model invocations. Pass the multi-provider off params (`techniques-small.md § Triggering thinking-off across providers`). Reserve `thinking=on` for tasks that genuinely need multi-step planning AND when the model is ≥7B AND when you can validate output is not truncated.

**When the user has a model card that says "supports thinking" or "reasoning model"**: that's a capability, not a recommendation. For compile-time tasks (extract / classify / NL→DSL / route), thinking-off is correct on 2-3B even if the card showcases thinking-on examples.

---

## 3. Self-verification on the same model

**The anti-pattern:**
```
# Pass 1
"Emit the action."

# Pass 2 — same model
"Re-examine your answer. Look for errors. Correct it if needed."
```

**Why it regresses small models:**
- Small models interpret "re-examine" / "double-check" / "look for errors" as **"find the error"** — confirmation anti-bias
- The model finds errors that aren't there and flips correct answers to wrong
- Same model, same training distribution → same blind spots → no error-detection capacity

**Reproduced regression** (`nl_rules_farm`, Ministral 3B):
- Pass 1: 78%
- Pass 2 with self-verify: 38% (−40pp)
- Diagnostic: 17 correct answers were flipped to wrong on pass 2; only 3 wrong answers were corrected — net catastrophic regression

**What to do instead:**
- **Worker + verifier**: small worker → large verifier. The verifier must be strictly stronger than the worker to break the same-blind-spot loop. See `techniques-small.md § Worker + verifier`.
- **Multi-pass on the same model only works for analyze-then-emit** patterns where pass 2 has different input (the analysis), not the same input + a "re-check" framing. See `techniques-small.md § 2-pass: analyze then emit`.

**Backing:** [arxiv:2404.17140](https://arxiv.org/pdf/2404.17140): "Small LMs Need Strong Verifiers."

---

## 4. Native tool calling on 2-3B without few-shot

**The anti-pattern:**
Switching from "emit JSON" output mode to OpenAI-style function calling via `tools` parameter, without providing few-shot tool-call examples in system.

```python
client.chat.completions.create(
    model="gemma-4-e2b",
    messages=[{"role": "system", "content": "You decide actions"}, ...],
    tools=[{"type": "function", "function": {...}}],  # ← schema only
    tool_choice="required",
)
```

**Why it regresses small models:**
- 2-3B models have limited training data for native function calling, especially for non-mainstream verbs
- Vendor docs explicitly warn — Microsoft Phi-4-mini card: "with function calling scenarios, the model could sometimes hallucinate function names or URL's"
- Switching from JSON to tools format strips few-shot demonstrations that were anchoring the model — the model has neither training-data fluency nor in-context demonstrations to fall back on
- Exception handling and error paths (which existed as examples in the JSON-mode prompt) silently disappear

**Reproduced regression** (`nl_rules_farm`, Gemma 4 e2b):
- JSON mode + 4 examples: 93%
- `tools` mode, same schema, no tool-call examples: 64% (−29pp)
- Failure breakdown: thinking-token leak in content, total loss of error-path handling

**What to do instead:**
- **If you don't need tools**: stay in JSON mode with `response_format: {"type": "json_object"}` or `json_schema` (per provider, see `matrix-small.md`).
- **If downstream framework requires tools**: stay in tools mode but add **3–5 few-shot tool-call examples in system** demonstrating each verb and the error path. The model needs in-context demonstrations to replace the missing training-data fluency.

**Backing:** Phi-4-mini model card (Microsoft, official); our reproduction on Gemma 4 e2b.

---

## 5. Blacklist of forbidden behaviors (long list)

**The anti-pattern:**
```
## Forbidden

- Do not use English words.
- Do not use these specific anglicisms: "плагин", "контент", "лайк", "пост", "контекст", "флоу", "пайплайн", "митап", "стрим", "челлендж", "роадмап", "фидбек", "тайминг", "перформанс", "deploy", "release", "feature", "build", "test", "fix", "bug", "patch"...
- Do not break character.
- Do not break the fourth wall.
- Do not use emoji.
- Do not use markdown.
- Do not capitalize the first letter.
- Do not include the speaker's name.
- ...
```

**Why it regresses small models:**
- Long lists of "don't do X" without "do Y instead" leave the model with no positive instruction to fall back to
- Each individual prohibition gets diluted by the list length
- "Don't think about a pink elephant" effect — naming forbidden items raises their salience
- 2-3B models can't keep a 20-item blacklist in working attention

**Backing:** [Pink Elephant paper](https://arxiv.org/abs/2503.22395): "open-source models exhibit 2.2× higher fragility than commercial counterparts, with negation-bearing syntax being the dominant failure mode, with some models endorsing actions at 80-97% rates even when asked whether agents should not act."

**What to do instead:**
- **Whitelist > blacklist.** State what IS allowed, not what isn't. "Use these 8 verbs: ... — anything else is `error`."
- For style/tone constraints, use a **style anchor**: 1-3 paragraphs in the target voice as a demonstration block. The model copies the demonstrated style rather than navigating a list of negations.
- Keep Forbidden to **3-5 named anti-patterns max**, each tied to a concrete failure mode (`techniques-small.md § Anti-defaults block`).

---

## 6. System prompt in the same language as the task input (for 2-3B)

**The anti-pattern:**
```
# system.md (Russian, for a Russian-input task on a small model)
Ты компилятор русского языка в DSL. Прочитай команду игрока и...
```

**Why it regresses small models:**
- Non-EN tokenization is 1.5-2× less dense than EN — system prompt eats more of the context budget for the same semantic content
- 2-3B models have less non-EN instruction-following training data → more brittleness on system instructions when those instructions are in non-EN
- Latency penalty: more tokens to process, same compute

**Reproduced gain** (`nl_dsl_game`, RU input):
- RU system + RU input: baseline
- EN system + RU input: +4-8pp pass rate, −35-60ms latency, zero regressions
- Verified across Gemma 4 e2b, Mistral 3B, Qwen 3.5 2B

**What to do instead:**
- **Default: EN system prompt** for 2-3B local models, regardless of input language.
- Input language and output language follow user need; system instructions are in English.
- If the model drifts to English in output, add a final-position language gate (`techniques-small.md § EN system unlocks non-EN performance`).

**Exception:** dedicated non-EN tunes (saiga, T-lite, RuGPT) have a shifted corpus balance; same-language system can match or beat EN. Verify before applying.

---

## 7. Frontier-style hidden-intent prose

**The anti-pattern:**
```
# system.md

You're working alongside the player as a thoughtful collaborator. They expect you to anticipate adjacent concerns, push back when they suggest something risky, and steelman their idea before agreeing. Default to warm prose, not bullets. Treat their request as a minimum — raise the next obvious move they didn't think of.
```

This is the **creative-domain kernel from `principles.md` § Pattern: creative-domain kernel for Opus 4.7**. It's load-bearing for Opus 4.7 in advisory roles, and it's a documented win on frontier models.

**Why it regresses small models:**
- "Anticipate adjacent concerns" / "steelman" / "push back" — these are hint-and-vibes phrases. 2-3B models are highly literal — they can't extrapolate from vibes the way Opus 4.7 can
- "Default to warm prose, not bullets" interpreted literally — model emits a long disclaimer paragraph instead of structured output
- "Treat request as minimum" — model starts inventing fields and actions not in the input (substitution bias triggered)
- Tone/role framing competes for attention with task-critical structural rules

**What to do instead:**
- **Strip hint-and-vibes framing entirely** for compile-time task prompts on 2-3B
- Replace with **task-facing skeleton** (`techniques-small.md § Skeleton: small-model task prompt`)
- If the agent's whole role is creative (chat companion, narrator), the small-model framing is different: heavy reliance on **style anchor** (1-3 paragraphs of target voice) rather than abstract directives. But for any task that has a defined structured output, drop creative-kernel blocks.

**Diagnostic question:** Does the model need to emit a structured output (JSON, DSL command, classification label)? If yes — strip the kernel. The kernel is for free-form prose generation, not compile-time tasks.

---

## 8. Persona block for compile-time tasks

**The anti-pattern:**
```
You are an expert Russian-to-DSL compiler with 10 years of experience translating game commands. You are precise, methodical, and never make mistakes.
```

**Why it regresses small models:**
- Persona claims ("expert", "never makes mistakes") don't change the model's actual capabilities — they raise the model's confidence in wrong answers
- "Methodical" / "precise" without operational definition is noise on 2-3B
- Eats attention budget that would otherwise go to whitelist / examples / Forbidden

**Note** — this differs from the frontier-side rule. On Gemini 3 a *functional* persona ("You are a planner") gives a +5% reasoning boost. On 2-3B local models, even functional personas mostly add noise. Skip persona blocks on small-model task prompts; if you must include one, keep it to a single functional sentence at top.

**What to do instead:**
- Open with task identity in one neutral sentence: "You decide one next action given rules and state."
- Let the procedure + whitelist + examples define what "expert behavior" looks like operationally.

---

## 9. Single-line "be concise / be exact / be careful" instructions

**The anti-pattern:**
```
Be concise. Be exact. Pay attention to negations. Follow the rules carefully.
```

**Why it regresses small models:**
- "Be concise" / "be exact" / "be careful" are abstract qualities without operational meaning. Model has no training signal for what these phrases should change in its output
- "Pay attention to X" reads as a hint that X is important but doesn't tell the model how to handle X
- Adding these lines dilutes attention to the concrete structural rules elsewhere in the prompt

**What to do instead:**
- Replace with operational instructions tied to format/structure: "Output one JSON object with exactly two fields", "Use only verbs from the whitelist above"
- For things like "be careful with negations" — replace with a few-shot example demonstrating the negation case
- For things like "be concise" — set a length constraint (`max 200 chars`, `one line only`, JSON schema with `maxLength`)

---

## 10. Mixing system role and first-user-turn instructions (Gemma)

**The anti-pattern:**
```python
# Gemma model (instruction-tuned, no system role support)
messages = [
    {"role": "system", "content": "You are a DSL compiler..."},
    {"role": "user", "content": "Compile this: атакуй врага"},
]
```

**Why it regresses small models (specifically Gemma family):**
- Gemma instruction-tuned models **do not support a system role** — they only have `user` and `model` turns ([Google docs](https://ai.google.dev/gemma/docs/core/prompt-structure))
- Different inference engines handle the `system` field differently: LM Studio silently merges it into the first user turn, llama.cpp may drop it, vLLM behavior varies
- Behavior is non-portable across providers → same prompt produces different outputs depending on which engine the user is on

**What to do instead:**
For Gemma family specifically — fold system-level instructions into the first user turn:
```python
messages = [
    {"role": "user", "content": "<system instructions here>\n\n<actual user input>"},
]
```

For other small-model families (Qwen, Mistral, Phi, Llama) — system role works fine; this anti-pattern is Gemma-specific.

**Detection cue:** if you see `{"role": "system", ...}` in a Gemma-targeted prompt, flag it. The current behavior might be working (silent merge) but it's fragile to a provider switch.

---

## 11. `temperature: 0` for all small-model calls

**The anti-pattern:**
```python
client.chat.completions.create(
    model="gemma-4-e2b",
    temperature=0,
    ...
)
```

**Why it can hurt small models:**
- `temp=0` (greedy decoding) on some small models causes **repetition loops** — the model picks the highest-probability token at every step, which can hit attractor states where token N → N+1 → N (loop)
- Especially common on Gemma family and on Phi-4-mini in long outputs
- Symptom: "the the the the..." or "carrot carrot carrot..." in output

**What to do instead:**
- **`temperature=0.1`** is a safer default for structured-output tasks — still nearly deterministic, breaks repetition loops, matches our project default
- For tasks where you genuinely need full determinism: `temp=0` + low `repeat_penalty` setting (1.1-1.2 on llama.cpp), AND test on a sample before committing
- Variance reminder: `temp=0.1` on 32-case suite gives ±10pp run-to-run. Any effect <10pp is within noise. See `FINDINGS.md`.

---

## 12. Long context with rules / examples in the middle

**The anti-pattern:**
```
<long pre-amble about the project>
<long history of past interactions>
<the rules — buried in the middle>
<more context>
<finally: user input>
```

**Why it regresses small models:**
- Small models lose attention on content placed in the middle of long contexts (lost-in-the-middle effect, well-documented)
- The rules that matter for the current decision get less attentional weight than the framing that doesn't
- 2-3B models have a tighter effective attention window than their nominal context window (128k context, ~8k attention focus)

**Reproduced behavior:** Google's docs explicitly say: "When working with large contexts, place your specific instructions at the end of the prompt (after the data context)."

**What to do instead:**
- Structure as: [bulky context at top] → [rules concisely] → [Forbidden] → [examples] → [user input]
- The "instructions sandwich" pattern (instructions both top AND bottom) works on frontier models but is wasteful on small models — the top copy gets ignored; just put it at the bottom
- For RAG-style tasks: put retrieved documents first, then the question/instructions at the end

---

## Cross-cutting note: variance and false signals

Many of the regressions above are documented at the level of `−10pp` or larger. On `temperature=0.1` over a 30-case suite, run-to-run variance is ±10pp. This means:

- **Effects smaller than 10pp are within noise** for single-pass measurements
- Always verify a "fix" with n≥3 runs, or switch to `temperature=0.0` for measurement (knowing it brings its own risks per #11)
- A "this technique improved my prompt!" claim without n≥3 evidence on the user's suite is anecdote, not signal

This applies in both directions — if the user reports "I added X and it broke things", confirm it's a real regression (n≥3 + statistical separation from baseline) before tearing out X.

When the user is making prompt changes without a benchmark suite, frame your recommendations as hypotheses: "this should help based on [reasoning], please verify with [smallest possible test]". Don't overclaim from theory alone.
