# Techniques for small local models — ready-to-paste snippets

Companion to [matrix-small.md](matrix-small.md). These are concrete snippets, structures, and patterns to copy and adapt when reviewing a prompt targeting a 2–9B local model.

Don't paraphrase from memory — copy from here and adapt to the user's domain. Every snippet has been observed to work on at least one production task in our `nl_rules_farm` / `nl_dsl_game` suites or is documented in vendor sources.

---

## Skeleton: small-model task prompt

This is the layout for a single-pass compile-time prompt (extract / classify / NL→DSL / route). Order matters — moving blocks around regresses performance on 2-3B even with identical content.

```
# <Task title — one line, what the model is>

You decide ONE <output type> given:

1. **<INPUT_KIND_A>** — short description, one line
2. **<INPUT_KIND_B>** — short description, one line

<2–4 sentences describing how to combine inputs to produce output. Avoid abstract principles — write the procedure as steps.>

## Output

Output a single <JSON object / line / structured form>. No prose, no markdown fence, no explanation.

Shape:
```json
{"field": "value"}
```

Examples of valid outputs:
```json
{"field": "value1"}
{"field": "value2"}
```

## <Whitelist / vocabulary section>

| Verb / category | Allowed values | Effect |
|---|---|---|
| ... | ... | ... |

## <Resolution procedure — numbered steps, 3–5 max>

1. ...
2. ...
3. ...

## Forbidden

- Do not <thing 1>.
- Do not <thing 2>.
- ...

## Examples

<4 examples mirroring known failure modes. See "Few-shot example template" below.>

---

Now read <input below> and emit <output>.
```

**Why this order works:**
- Task identity at top → model frames itself before reading rules
- Output format before whitelist → model knows shape before being shown contents
- Whitelist/vocabulary before procedure → procedure can reference vocabulary
- Forbidden before examples → the "anti" frame primes attention to format violations the examples will demonstrate
- Examples LAST → directly precede user input, maximum attentional weight
- Trailing "now read..." → primes the model to expect input, not more instructions

**Why each block deserves to be there:**
- Skipping vocabulary → model invents synonyms (substitution bias)
- Skipping Forbidden → model adds explanation prose
- Skipping examples → model improvises format on edge cases (negation, exception, error path)

---

## Few-shot example template — mirror failure modes

The golden technique for 2-3B. **Each example demonstrates one failure class** the model would otherwise hit. Examples are wrapped in `---` separators for visual segmentation (helps small-model attention).

```
---

Example 1 — <name the class this example demonstrates, e.g., "no-X precondition fires">.

Input:
\```
<input as the model will receive it, exact format>
\```

Output: `{"field": "value"}`

---

Example 2 — <next failure class, e.g., "adjective filter fails — must not fire">.

...

---
```

**Mandatory failure classes for any rule-following task** (omit those that don't apply):

| Example name | Demonstrates |
|---|---|
| `<simple positive case>` | Baseline — the model knows what success looks like |
| `<precondition fires>` | "If you have no X, do Y" — small models systematically skip negation preconditions |
| `<adjective filter rejects>` | "ripe X" rule doesn't fire on "growing X" — must produce `wait` / `null` / `no-op` |
| `<exception clause blocks default>` | "Always do X **except** when Y" — demonstrates exception path |
| `<out-of-whitelist → error>` | Rule references unknown item → emit error, **don't substitute** |
| `<implicit negation: "but not">` | "do X but not Y" — the highest-difficulty class even for e4b |

Count target: **4 for Gemma, 5 for Qwen, 4–5 for Ministral**. Going to 7+ examples doesn't continue to help and pushes the prompt past the model's effective attention window.

---

## Anti-pattern named block — Qwen / Ministral pattern

For models with documented substitution / rule-mutation bias (Qwen 2-4B especially), add an **anti-pattern block before Forbidden** with named failure modes. Naming the failure mode beats abstract principles.

```
## CRITICAL: <task category> are sacred

The <input category> are **input data**, not suggestions. You must:

- **Never paraphrase a <category>** to make it easier to satisfy.
- **Never substitute** an item / entity / action the input mentions with a similar in-whitelist alternative. If the input names `<example>`, treat it as `<example>` literally — don't silently swap it for `<similar>` because they look related.
- **Never skip a clause** because it would make a later <category> applicable. "If <precondition>, do X" must be checked against the state — if precondition holds, the clause fires; you do not fall through.
- **Never invent <action>** that the inputs don't authorise. If nothing fits, the answer is `<safe default>` — not "the most useful thing I can do".
```

**Why named anti-patterns work** where abstract principles don't: the model has a concrete failure mode to *avoid*, not a positive instruction to *follow*. "Never substitute" + named example is grippable; "be precise" is not.

Source: our `system_qwen.md` lifted Qwen 3.5 2B from 71% to 79% in `nl_rules_farm` with this block added.

---

## Markdown-tolerant output (Ministral pattern)

Mistral family emits markdown fences as a natural output style. Suppressing the behavior takes 5+ prompt iterations and still fails ~10% of the time. **Cheaper fix: allow fences, parse them out.**

In system prompt:

```
## Output

Output a single JSON object. You may wrap it in ` ```json ... ``` ` fences if it helps you produce well-formed JSON — the parser tolerates fences. Do NOT output prose, narration, or chain-of-thought.

Shape:
```json
{"field": "value"}
```
```

In the parser (downstream of the API call):

```python
def extract_json(text: str) -> dict:
    text = text.strip()
    if text.startswith("```"):
        # Strip ```json or ``` opening, ``` closing
        text = re.sub(r"^```(?:json)?\n", "", text)
        text = re.sub(r"\n```$", "", text)
    return json.loads(text)
```

Source: `system_ministral.md` lifted Ministral 3B from 79% to 86% in `nl_rules_farm` after switching from "no markdown fences" to "fences allowed, parser tolerates".

---

## EN system unlocks non-EN performance (2-3B)

For tasks where input is non-English (RU, DE, JP) but the model is a base 2-3B (Gemma 4, Qwen 3.5, Mistral 3B):

**Write the system prompt in English. Always.** Even when the input text and expected output text are in another language.

Why it works:
- EN tokenization is 1.5–2× denser than RU/DE for the same semantic content → more attention budget free for task logic
- Small models have more EN instruction-following training data than non-EN
- Latency improves by 35–60ms on the same hardware

Our measurement: on `nl_dsl_game` (RU input) switching system from RU to EN gave +4-8pp across Gemma 4 e2b, Mistral 3B, Qwen 3.5 2B with **zero regressions**.

Output language stays whatever the task needs — the system instructs in English, the model emits in the user's language. If the model drifts to English in output, add a final-position language gate:

```
## Output language

Respond in <Russian / German / etc.>. The system prompt is in English for clarity, but your output must be in <target language>.
```

**Exception**: skip this technique when the model is a dedicated non-EN tune (saiga, T-lite, RuGPT, *-it-2.1) — these have shifted the corpus balance and a same-language system can match or beat EN.

---

## 2-pass: analyze then emit (Ministral-specific win)

The only multi-pass pattern that consistently wins on a small model. **Works on Mistral family; regresses Gemma and is neutral on Qwen.** Verify before applying.

Pass 1 (analyze):
```
Read the rules and state below. For each rule, say whether its condition is currently true or false.

Output format — one line per rule:
Rule 1: <true / false> — <one-phrase reason>
Rule 2: <true / false> — <one-phrase reason>
...
```

Pass 2 (emit), with Pass 1 result included in user message:
```
Based on your analysis above, emit the single action authorised by the highest-priority rule whose condition is true.

If all rules are false, emit `<safe default>`.
If a rule asks for something outside the whitelist, emit `<error>`.

Output: single JSON object, no prose, no fences.
```

Our finding: Ministral 3B 78% → 88% on `nl_rules_farm` (`+10pp`). Latency cost: ~3× single-pass. Worth it for compile-time / batch tasks, not for per-tick / real-time loops.

**Why this fails on Gemma**: without `--jinja` in llama-server, Gemma 4 thinking-token leak contaminates the Pass 1 analysis output. Even with `--jinja` the gain is marginal. Stick to single-pass on Gemma.

---

## Worker + verifier (small worker + large verifier)

For when single-model + few-shot has hit its ceiling but you don't want to commit to using the large model on every tick.

Architecture:
- **Worker** (small, fast): single-pass emit
- **Verifier** (large, accurate, called only when worker is uncertain or the input class is known-hard): check worker's output against rules + state, ok / redraft

Trigger conditions for verifier (any of):
1. Worker emits a `wait` or `error` (rule possibly missed)
2. Input contains negation markers ("but not", "except", "kроме", "no X")
3. Confidence flag from worker (some models can return self-uncertainty)
4. Random sample (e.g., 5% of tickets for QA telemetry)

Snippet — verifier prompt:
```
You are a verifier. The worker model emitted <output> for the following input. Check whether this output strictly follows the rules and state below.

Rules:
<rules>

State:
<state>

Worker's output: <output>

Respond with one of:
- `{"verdict": "ok"}` — worker's output is correct
- `{"verdict": "redraft", "reason": "<one phrase>", "correct_output": <correct JSON>}`
```

Backing: [arxiv:2404.17140](https://arxiv.org/pdf/2404.17140) — small models can achieve significant improvement after 1 round of self-correction **when paired with a strong verifier**.

Cost: latency adaptive — simple cases stay fast (worker solo), hard cases pay verifier cost. Typically 90/10 split — net latency only ~1.3× worker.

---

## Compile-once decompose (neuro-symbolic split)

When the user's domain involves complex rules with negation, implicit exceptions, or compound conditions — and the small model systematically fails on those classes — the highest-leverage move is to **eliminate the failure class at compile time** rather than at inference time.

Architecture:
- **Compile stage** (large model, runs once when rules change): translate NL rules into structured atomic rules with no negation, no compound conditions, no implicit exception clauses. Output is a JSON tree the small model just needs to evaluate.
- **Inference stage** (small model, runs every tick): evaluate the structured rules against state. Pure structural matching — no NL parsing needed.

Compile prompt skeleton (for the large model):
```
Translate the following natural-language rules into structured atomic rules in this schema:

{
  "rules": [
    {
      "id": "<short>",
      "when": { "all_of": [...], "any_of": [...], "none_of": [...] },
      "then": { "action": "<verb>", "arg": "<arg>" },
      "priority": <int>
    }
  ]
}

Rewrite negations as positive conditions in `none_of`. Rewrite "X but not Y" as one rule with `all_of: [X_condition], none_of: [Y_condition]`. Never emit `not` inside a condition body.

Rules:
<NL rules>
```

The small model then sees only structured rules — no implicit negation to fail on, no compound exceptions, no NL parsing.

Backing: [Req2LTL](https://arxiv.org/html/2512.17334), [Decompose-and-Formalise](https://arxiv.org/html/2601.19605v1).

When to invest in this: the rule set changes rarely (compile cost amortizes), the tick rate is high (saves inference cost), and the failure classes are negation-heavy (the small model's weakest axis).

---

## Per-model system override (file layout)

When you have one task with one default system but one or two specific models that need targeted adaptations:

```
suites/<task>/
├── system.md            # default — works for most models
├── system_<model_a>.md  # adapts for model A's failure modes
└── system_<model_b>.md  # adapts for model B's failure modes
```

Each `system_<model>.md` is a complete drop-in replacement, not a diff or overlay. Add the model-specific anti-pattern block, swap example ordering if needed, but don't fragment by leaving partial files.

In code (provider-agnostic):
```python
def system_for(model: str) -> str:
    candidate = f"suites/{task}/system_{model}.md"
    if Path(candidate).exists():
        return Path(candidate).read_text()
    return Path(f"suites/{task}/system.md").read_text()
```

**Cite this pattern when the user has 3+ models, one task, and divergent failure modes.** Don't propose per-model `if/elif` branches in business logic — infrastructure-mapping (model name → file) is fine; behavior branches in code aren't.

---

## Structured output adaptation per provider

Small-model structured output works differently across local providers. The same `json_schema` constraint can lift one model 5pp on one provider and break it entirely on another.

| Provider | `json_object` (free JSON) | `json_schema` (validated) | GBNF grammar |
|---|---|---|---|
| LM Studio (1.x) | works | works (full Pydantic-like schema) | not exposed |
| llama.cpp `llama-server` | works | **fails** ("Failed to initialize samplers") on schemas with `properties`/`required` | works (write `.gbnf` file) |
| vLLM | works | works | works |
| Ollama | works (limited) | works (limited) | not exposed |

Practical adaptation:
- If targeting LM Studio: full `json_schema` with `properties`, `required`, type unions — fine.
- If targeting llama.cpp: degrade `json_schema` to `json_object`, OR write a `.gbnf` grammar OR move the schema constraint into the prompt body (with examples).
- If user is provider-agnostic: stick to `json_object` mode + 3–4 few-shot examples showing the exact JSON shape. This is the lowest-common-denominator that works everywhere.

---

## Triggering thinking-off across providers

Every small-model provider has its own knob for "disable native thinking". Reasoning-token leak in `content` is a major regression source on 2-3B (truncated output, format breakage).

Pass **all** these params in the API call — each provider takes its own and ignores the rest:

```python
extra_body = {
    "reasoning_effort": "none",                            # LM Studio
    "chat_template_kwargs": {
        "enable_thinking": False,                          # Qwen 3 family
        "thinking": False,                                 # Gemma alt
    },
    "reasoning": {"enabled": False},                       # OpenRouter
    "thinking": False,                                     # generic
}
```

For llama.cpp specifically: launch the server with `--reasoning-format none` so thinking blocks stay in `content` as visible `<think>...</think>` tags (then strip in post). Without this flag, reasoning gets routed to the separate `reasoning_content` field which most OpenAI-compatible parsers ignore — looks fine in API response but truncates the actual answer.

Observed safety: passing all params at once is safe — providers ignore unknown fields. The only risk is your downstream JSON-mode also imposing a separate `<think>` block (Qwen's `enable_thinking=False` inserts an empty one to prime the no-thinking path) — verify the post-strip doesn't break on it.

---

## Anti-defaults block — name what NOT to do

For prompts where the model has known habitual patterns to break (verbose intro, scaffolding, repetition), name them explicitly. Generic "be concise" doesn't work on 2-3B.

Effective block:
```
## Forbidden

- Do not output explanations, prose, or markdown fences.
- Do not invent verbs / fields / categories not in the whitelist above.
- Do not emit more than one <unit> per response.
- Do not narrate <thing> — only the JSON object.
- Do not include reasoning or chain-of-thought in the output.
```

Each line names ONE concrete thing. Position: **directly before examples**. The examples then demonstrate what the absence of those patterns looks like.

**Why this works**: small models pattern-match named prohibitions to their nearest training-data analog. "Do not narrate" reads as a recognizable pattern; "be concise" doesn't have a corresponding training signal.

---

## Length-control techniques (max_tokens won't save you)

Small models will happily emit garbage to fill a `max_tokens` budget, then truncate the actually-needed output. To prevent:

1. **Explicit length cap in prompt**: "Output exactly one JSON line under 200 characters."
2. **Format constraints that imply length**: a JSON schema with fixed required keys and value type constraints (`string` with `maxLength`) naturally caps output.
3. **Examples at the desired length** — model copies example length. If your 4 examples are 30-50 chars each, the model targets that range.

`max_tokens` should be set to **2× the longest expected output**, no more. Setting it to 4096 "just in case" with a small model often *causes* the rambling you're trying to prevent (thinking models will fill it; non-thinking models will sometimes emit explanation prose to use the budget up).

---

## When to fail over to a fine-tune (LoRA / SFT)

Prompt-tuning has hit ceiling when:

1. You've applied few-shot + named anti-patterns + per-model override → still missing one specific failure class
2. The failure class is well-defined (e.g., "implicit negation in Russian", "specific DSL grammar rule")
3. You have ≥50 examples of the failure class with correct outputs
4. The base model is in the 2-7B range (LoRA economical here)

At that point, recommend the user create a small SFT dataset targeting the failure class and apply LoRA, rather than further prompt iteration. See project AGENTS.md § "LoRA fine-tuning" for the loop.

**This skill stops at prompt-tuning** — don't propose LoRA mechanics, only signal the threshold where further prompt work has diminishing returns.
