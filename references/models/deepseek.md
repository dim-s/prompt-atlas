# Model-specific wording differences — DeepSeek family

What changes about how you should PHRASE prompts for DeepSeek V4-Pro / V4-Flash, V3.2 / V3.2-Speciale, and the R1 lineage. Companion to `claude.md`, `gpt.md`, `gemini.md`, `kimi.md`, `glm.md`, `qwen-frontier.md`.

DeepSeek is the most opinionated of the current frontier vendors in **where to put instructions** — and its opinion is opposite to most other vendors. This file leads with that rule because the cost of getting it wrong is high.

---

## Family-wide rules (apply to all current DeepSeek versions)

These hold across V3.2 → V4-Flash → V4-Pro.

### 1. Core instructions live in the USER prompt, not the system prompt — DeepSeek-specific

DeepSeek's own guidance (V4 practitioner guides explicitly cite the R1+V4 documented behavior) is to **place core instructions in the user prompt**, keep the system prompt brief, and treat the user prompt as the binding contract.

**This is opposite to most other vendors.** Claude, GPT-5.x, Gemini 3, Kimi K2.6, and frontier Qwen all expect the bulk of instruction in the system prompt. DeepSeek inverts this.

| Other vendors | DeepSeek V4 |
|---|---|
| **System prompt:** "You are a senior backend engineer. Follow these rules: [50 lines of constraints, examples, formatting, edge cases]. Use these tools when..." | **System prompt:** "You are a senior backend engineer." |
|  | **User prompt:** "[Task description.] Constraints: [...]. Examples: [...]. Format: [...]. Use these tools when..." |

Wording mitigation when porting Claude / GPT prompts to DeepSeek V4:

- **Move the bulk of the system content into the user message.** Keep a single-line functional role in the system slot.
- **For role-play and thinking-mode work specifically:** put instructions at the **end of the first user message** — that's the position where DeepSeek's instruction-following is most stable (per V4 practitioner guides, this matches the model's training-data layout).
- **Don't fight this with workarounds** like "in the system prompt, the rules are: X" referenced from the user prompt — V4 will ignore complex system-prompt content.

Quantitative claim from V4 prompting guides: **system prompt overuse causes ~85% of common DeepSeek V4 errors.** Treat the rule seriously.

**Agentic failure mode.** A tool-using DeepSeek agent that carries its rules and search methodology in the system prompt tends to **fabricate confident false negatives** on absence questions ("X doesn't exist anywhere" for something that does) and mis-scope its searches — the under-weighted system methodology yields a shallow search that then asserts. Moving the bulk to the user turn (system kept to a one-line role) fixes it on the same model and tools — an instruction-authority effect, not a capability gain. Symptom → diagnosis: a confident "doesn't exist anywhere" from a DeepSeek agent → suspect system-prompt overuse and verify before trusting the negative.

### 2. Brief system prompt + structured user prompt = the canonical V4 pattern

The canonical pattern V4 guides recommend:

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

Note the components:

- **System:** one-line functional role
- **User `<context>`:** structured data with explicit `relevance` tags (V4 uses this for attention prioritization)
- **User `<instruction>`:** the actual task, with output shape demanded in prose **even when** `response_format` is set at the API level

This XML-tagged context structure has measured ~92% task success vs ~45% for unstructured dumps (per practitioner-guide benchmarks). The 47-point gap is meaningfully larger than the XML-vs-markdown gap on other vendors.

### 3. `reasoning_content` must be round-tripped on multi-turn — V4 API breaking change

V4 returns a `reasoning_content` field alongside the response content. On subsequent turns, this field **must be included** in the message history or the API returns 400.

R1 and V3.2 **rejected** `reasoning_content` on input (the reverse) — so multi-turn flows that worked on V3.2 break on V4 in a non-obvious way.

This is an API/SDK concern, not a prompt-wording concern. But it's worth flagging in reviews when:
- The artifact is a multi-turn agent prompt being migrated V3 → V4
- The user reports "DeepSeek throws 400 errors on the second turn"

Wording-side: don't reference `reasoning_content` in prose. The behavior is a serialization concern, not something the model needs prompted about.

### 4. Tool calls in thinking mode — unlike R1

V4 supports tool calls while reasoning. R1 forced a choice: reason **or** call tools. V4 unifies them.

Wording implication: agentic loops can mix reasoning and tool calls in the same response without prompt-side workarounds. Prompts that previously instructed "first reason about the task, then in a separate response call the tool" can be simplified.

### 5. Effort-model thinking lever: `thinking` toggle + `reasoning_effort` (`low` / `high` / `max`)

Since the **V4-Pro GA update (2026-08-13)** DeepSeek exposes reasoning depth as an **effort model**, like the other frontier vendors — the old three-level `thinking: "off" | "high" | "max"` parameter set is **obsolete**:

| Format | Thinking toggle | Effort control |
|---|---|---|
| OpenAI (Chat Completions) | `extra_body={"thinking": {"type": "enabled/disabled"}}` | `reasoning_effort` (`low`/`high`/`max`) |
| Anthropic | `reasoning.effort` (`none`/`low`/`high`/`max` — `none` disables thinking) | same field |
| Responses API | — | `output_config.effort` (`low`/`high`/`max`) |

**Thinking is enabled by default, default effort `high`.** Actual mapped effort: requested `low` → `low`; `medium` → `high`; `high` → `high`; `xhigh` → `high`; `max` → `max` (identical for V4-Pro and V4-Flash).

Wording-side: same rule as other vendors — don't write "think step by step" in the body, and don't ask the model to *lower* its own reasoning via prose. Set the parameter. A "don't think / answer immediately" line is now a **reduced-reasoning** request (set effort `low`), not an off-switch equivalent — and on the Anthropic surface `none` is the only true disable.

### 6. Demand JSON in the user prompt even with `response_format` set

V4-specific: even when the API call includes `response_format={"type": "json_object"}`, the user prompt must still explicitly demand JSON output. The combination of "API parameter" + "prose demand" is the load-bearing pattern; either alone is unreliable.

V4 guides estimate missing-JSON-instruction as ~60% of structured-output errors.

The cleanest pattern in the user prompt:
```
... Output strictly as JSON matching this shape: {field1: ..., field2: ...}. No prose outside the JSON.
```

This costs little on other vendors (mostly redundant) and is load-bearing on V4. Cross-vendor wording should default to this even when the target is non-DeepSeek.

### 7. Slightly more literal than Claude

V4 follows instructions more literally than Claude. Prompts that depend on Claude's "infer my intent" tolerance need to be made explicit when porting to V4. This is directionally similar to GPT-5.5 and Opus 4.7 — but a step more literal than Opus 4.7.

When porting a Claude-tuned prompt:
- Spell out scope: "modify only the function `validateInput`" instead of "modify the validation logic"
- Spell out boundaries: "do not refactor anything outside this function" — V4 honors this without overtriggering
- Spell out success criteria: "the existing tests pass plus a new regression test you add"

### 8. Maximum context > minimum tokens — DSA economic inversion

V4's DeepSeek Sparse Attention achieves ~27% of per-token FLOPs and ~10% of the KV cache footprint of V3.2. **Cached input tokens cost $0.028 per million** — roughly 100× cheaper than other vendors' cached input.

The economic implication inverts the historical "compress your prompt" pattern: with V4, **front-loading static documentation and few-shot examples for cache hits is cheap and rewarded**. Prompts that have been aggressively compressed for token economy can be expanded without cost concern.

Wording implication: don't ask "is this too long?" — ask "is this stable enough to be cached?" Stable front matter is better than terse front matter.

### 9. 1 M context nominal, reasoning degrades past ~500 K

V4 advertises a 1 M token context window. Practitioner reports note **reasoning quality degrades past ~500 K tokens**, though retrieval remains viable up to ~750 K at cache-hit rates >70%.

Wording implication:
- For reasoning tasks (the model needs to derive conclusions from the context): keep under 500 K
- For retrieval tasks (pull the right passage and quote it): up to 750 K acceptable
- For both at scale: chunk into stages — retrieve first, then reason on the chunked retrieval

### 10. No standard HuggingFace chat templates — use model-specific encoding

V4 lacks generic Jinja chat templates. Self-hosted setups need DeepSeek-specific encoding scripts. This is a deployment-side concern, but worth flagging in reviews when the artifact is a prompt being applied across multiple inference engines — confirm with the user that they're using DeepSeek-aware tooling.

---

## DeepSeek V4-Pro (April 24, 2026 — current frontier; GA update 2026-08-13)

### Headline facts

- **Architecture:** 1.6 T total parameters, 49 B activated per token, MoE (reported figure — not restated in the 08-13 launch post)
- **Context:** 1 M tokens (with reasoning degradation past ~500 K)
- **Training:** 32 T tokens
- **GA update (2026-08-13, `deepseek-v4-pro` name unchanged):** released out of preview on APP / Web / API; notable agentic gains — HLE 42.7 (w/o tools) / 60.0 (w/ tools), Terminal-Bench 2.1 **87.9**, DeepSWE **62.7** (up from 12.8 on the preview), CyberGym 83.3, Toolathlon-Verified 74.1, AutomationBench (public) 31.8; Artificial Analysis moves V4-Pro from 45 → 53 on its Intelligence Index (secondary reporting)
- **Native OpenAI Responses API support**, with a one-click Codex configuration — the Responses-API surface and its `output_config.effort` join the existing Chat Completions / Anthropic surfaces
- **Thinking is effort-model since this update** — see family rule #5; default enabled with effort `high`
- **Pricing switched to peak/off-peak from 16.08 16:00 UTC** — off-peak is half of peak; several tariffs rose sharply (input+output up to ~2–3.7×; cache-hit input from +52% up to **+1100%**), so the cached-prompt economics that the atlas's DSA rule (#8) relies on are now time-of-day dependent and measurably more expensive
- **Available via:** DeepSeek's hosted API, self-hosted (open weights)

### Wording behaviors specific to V4-Pro

- **All family rules** (user-prompt priority, XML-tagged context, JSON-in-prose) apply most strongly here
- **Tool-use gap vs the closed flagships has narrowed but not closed** with the 08-13 GA — V4-Pro 0813 (AA Index 53) still trails the top closed models on long-horizon agentic coding (Opus 5 / GPT-5.6 Sol / Grok 4.6). For agentic loops where tool reliability dominates, the closed frontier may still be the right target; for pure reasoning + tool use, V4-Pro is competitive at a fraction of the cost.

### When V4-Pro specific tuning helps

- Complex reasoning workloads where DSA cost-efficiency is the actual win
- Coding tasks scoped tightly (specific function, specific bug fix) — V4-Pro's literalism is a strength here
- Long-context retrieval (under 500 K reasoning, under 750 K retrieval)

### When NOT to invest in V4-Pro specific tuning

- Long-horizon autonomous agents — Opus 5 / GLM-5.3 / Qwen3.8-Max / GPT-5.6 Sol all lead on this axis
- Broad factual recall — V4-Pro abstention isn't as high as Qwen but factual accuracy lags closed flagships
- Visual / multimodal — V4 is text-only

---

## DeepSeek V4-Flash (April 24, 2026 — official release 2026-07-31)

### Headline facts

- **Architecture:** 284 B total parameters, 13 B activated per token
- Same 1 M context, same DSA, same family character as V4-Pro — at a much lower cost / latency profile
- **Official release 2026-07-31** (`deepseek-v4-flash` name unchanged; the `-0731` build is the same architecture as the Preview, re-post-trained) — Terminal-Bench 2.1 82.7, DeepSWE 54.4, CyberGym 76.7, Toolathlon 70.3 (DeepSeek's own harness figures)
- **Self-host viable** at a much smaller hardware footprint than V4-Pro

### Wording adjustments from V4-Pro → V4-Flash

- **More granular spec needed** — Flash is less forgiving of vague constraints
- **Stronger user-prompt priority** — the smaller model leans harder on the user instruction
- **Tighter context budgets** — even though nominal context is 1 M, Flash reasoning degrades earlier than Pro

### When V4-Flash specific tuning helps

- Cost-sensitive production agents
- High-volume batch tasks
- Latency-critical interactive UIs where V4-Pro's reasoning depth isn't needed

---

## DeepSeek V3.2 / V3.2-Speciale (February 15, 2026 — legacy)

### Headline facts

- **Released:** February 15, 2026 — "fundamentally shifted the competitive landscape" at the time
- **V3.2-Speciale:** high-compute variant
- **Architecture predates DSA** — historical token-cache economics apply (no $0.028/M caching)
- **Multi-turn API:** `reasoning_content` was **rejected** on input — the reverse of V4

### When V3.2 specific tuning matters

- Production stacks still on V3.2 that haven't migrated to V4
- Cost-sensitive deployments where V3.2 pricing wins (V4 may be cheaper now, but historic deployment lock-in matters)

### Migration V3.2 → V4

Primary breaking changes:
- **API:** `reasoning_content` round-trip behavior inverts. Multi-turn flows that worked on V3 silently break on V4 (or vice versa).
- **Prefix cache hashing:** differs between V3.2 and V4 — caches don't transfer.
- **Tokenizer specifics:** if you hard-coded V3.2 tokenizer assumptions, audit before the port.

Wording-side, the rules don't change much (user-prompt priority, XML context structure, JSON-in-prose all worked on V3.2 and continue to work on V4). The migration cost is API-level, not prompt-level.

---

## DeepSeek R1 (lineage note)

R1 was the reasoning-first variant in the V3 era. Key historical facts that affect prompt-atlas reviews when the user mentions R1:

- **Tool calls and reasoning were mutually exclusive** — V4 unified them
- **`reasoning_content` was rejected** on multi-turn input (same as V3.2)
- **R1-era prompts** that worked around the tool/reason exclusion can be simplified on V4

If you encounter a prompt that explicitly says "first reason, then in a follow-up call use the tool" and the target is V4, flag it as obsolete and simplify.

---

## Cross-model wording principles (DeepSeek side)

Mirror of the family-wide section, distilled for cross-model reviews:

- **User-prompt priority** (opposite to every other vendor) — DeepSeek's signature divergence
- **Brief system prompt** — system overuse is the #1 failure mode
- **XML-tagged context with `relevance` attrs** — 92% vs 45% accuracy gap
- **Demand JSON in prose AND set `response_format`** — both, not either
- **Effort-model thinking lever** — `thinking` toggle + `reasoning_effort` `low`/`high`/`max` (default enabled, effort `high`; Anthropic surface adds `none` as the disable); the old `off`/`high`/`max` triplet is obsolete since 2026-08-13
- **Tool calls work in thinking mode** — V4 unified the lineage
- **`reasoning_content` round-trip on V4 multi-turn** (API breaking change from V3)
- **Slightly more literal than Claude** — spell scope, boundaries, success criteria
- **Cache-hit economics** invert compression — front-load stable content
- **1 M nominal, ~500 K reasoning ceiling** — split retrieval and reasoning stages for very long context
- **No standard HF Jinja templates** — DeepSeek-specific encoding required

---

## Cross-vendor rules (when DeepSeek is one of several targets)

DeepSeek's user-prompt-priority rule is the **load-bearing cross-vendor question**. If a single prompt must run on DeepSeek V4 **and** Claude / GPT / Gemini / Kimi / GLM / frontier Qwen:

### Option A — write for DeepSeek, accept slight overhead on others

Move the bulk of instruction into the user message. Other vendors will read it from there with no penalty (they tolerate instruction in either system or user; just slightly prefer system). Cost: a leaner system prompt may look "incomplete" in tools that surface system content to humans, but the actual model behavior is unaffected.

### Option B — write for the others, accept worse DeepSeek behavior

Bulk in system. Acceptable if DeepSeek is a tertiary target and you're tuning primarily for Claude / GPT / Gemini. Expect ~85% of DeepSeek failures attributable to this.

### Option C — maintain a DeepSeek-specific override

If the artifact is `AGENTS.md`, ship an `AGENTS.deepseek.md` override that replaces the system bulk with a brief role and moves the rest into a template injected into the user message.

**Recommended default:** Option A unless the user has explicit reason to favor others. The cost of Option A on non-DeepSeek vendors is small; the cost of Option B on DeepSeek is large.

### Cross-vendor wording table addendum (DeepSeek-specific)

| Axis | Direction on DeepSeek V4 | Cross-vendor compromise |
|---|---|---|
| Instruction placement | **user prompt** (system stays brief) | move bulk to user; system is one-line role |
| XML context structure | helps strongly (92% vs 45%) | use XML; works on Claude / GPT / Gemini / Kimi too |
| JSON-in-prose demand | required (with `response_format` redundant safety) | demand JSON in prose universally — costs nothing on other vendors |
| Cache-hit front-loading | helps strongly ($0.028/M) | works on all vendors, just less dramatically |
| Tool-use in thinking | works | safe on V4; other vendors don't have this conflict |
| `reasoning_content` round-trip | mandatory | client-library concern, not wording |

---

## Source notes

DeepSeek V4 documentation and practitioner guides are still maturing. The behaviors documented here come from:

- DeepSeek's official API changelog ([api-docs.deepseek.com/updates](https://api-docs.deepseek.com/updates/), read 2026-08-18) — the 2026-08-13 V4-Pro GA entry (effort levels `low`/`high`/`max`, Responses API + Codex adaptation, peak/off-peak pricing from 16.08), and the 2026-07-31 V4-Flash official release
- DeepSeek's official thinking-mode guide ([api-docs.deepseek.com/guides/thinking_mode](https://api-docs.deepseek.com/guides/thinking_mode)) — the OpenAI / Anthropic / Responses API parameter formats and the effort mapping table; **"thinking mode is enabled by default, with the default effort being `high`"**
- Independent pricing reporting (computerworld.com, 2026-08) — cache-hit tariffs up +52% to +1100%; off-peak = half of peak
- Datanorth's V4-Pro-0813 coverage (datanorth.ai, 2026-08-14) — GA date, benchmark context (AA Index 53), Harness v0.1 open-sourced under MIT
- DeepSeek V4 practitioner guides (deepseekai.guide, skywork.ai), the CodersEra V4 vs V3.2 comparison, the BentoML "Complete Guide to DeepSeek Models"

The strongest claims — **user-prompt-priority** and the **`reasoning_content` round-trip** — are documented across multiple practitioner sources and are repeatable. Treat them as load-bearing. Other axes (cache economics, 1 M context with 500 K reasoning ceiling) are more speculative and worth measuring before committing to a high-stakes prompt design.

DeepSeek does not (as of May 2026) publish an official "prompt best practices" document in the way Anthropic and OpenAI do — guidance comes from the model card, release notes, and community practitioner guides. Treat this file as the synthesis it is.
