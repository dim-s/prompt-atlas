# Model-specific wording — Google Gemini 3.x

What changes about how you should PHRASE prompts for Gemini 3.x. Companion to `claude.md` (Claude) and `gpt.md` (GPT-5.x). Focused on wording — not API parameter values, pricing, or context limits beyond what affects the prompt.

When the artifact runs in Gemini CLI specifically (GEMINI.md, AGENTS.md via fileName config, Gemini CLI subagent, skill, slash command), also read `../agentic-systems/gemini-cli.md` — the CLI layers its own behavior on top of the underlying model.

---

## Family-wide rules (apply to all Gemini 3.x versions)

These hold across 3.0 → 3.1. Version-specific notes follow below.

### 1. Reasoning model — concise prompts win

Google's official prompting guide is explicit: *"Gemini 3 is a reasoning model. Be concise in your input prompts. Gemini 3 responds best to direct, clear instructions and may **over-analyze** verbose or overly complex prompt engineering techniques used for older models."*

Practical impact: prompt-stacks ported from Gemini 2.5 (or worse, from older GPT/Claude) with extensive scaffolding will **underperform** versus a leaner zero-baseline rewrite.

This is the same family-wide pattern as GPT-5.5 and Claude Opus 4.7: frontier reasoning models prefer concise prompts. The patches accumulated against earlier-version quirks now address problems the new model doesn't have.

### 2. Chain-of-thought scaffolding becomes harmful

Google's migration guide: *"If you were previously using complex prompt engineering (like chain of thought) to force Gemini 2.5 to reason, try Gemini 3 with `thinking_level: 'high'` and simplified prompts."*

When porting a 2.5-tuned prompt: **strip CoT scaffolding entirely**. Don't leave it "just in case" — internal thinking happens automatically on Gemini 3, and explicit CoT competes with it.

When reviewing a Gemini 3 prompt that says *"think step by step"*, *"first do X, then Y, then Z"*, *"break this down"* → flag as `[IMPROVE]` and recommend deletion + raise `thinking_level` if reasoning is shallow.

### 3. Temperature — DON'T tune (Google's strong recommendation)

> *"For Gemini 3, we strongly recommend keeping the temperature parameter at its default value of 1.0. Changing the temperature (setting it below 1.0) may lead to unexpected behavior, looping, or degraded performance, particularly with complex mathematical or reasoning tasks."*

This is **opposite to Claude and GPT-5.x** where temperature is freely tunable. When reviewing a Gemini prompt:
- If the prompt body mentions temperature → flag for removal (don't reference API params in prompt body anyway)
- If the surrounding code sets `temperature < 1.0` → flag as a non-wording finding ("setting fix, not wording change")
- For cross-vendor `AGENTS.md`: don't reference temperature at all; let each vendor's API config handle it

### 4. Identity-based prompting — `+5%` reasoning boost

Google's research: *"Identity-based prompting — telling the model what kind of thinker it is — boosted performance by around 5% on complex reasoning tasks."*

Effective patterns:
- *"You are a planner."*
- *"You must analyze before responding."*
- *"You should evaluate risk and reorder steps."*

**This is the OPPOSITE default to GPT-5.5**, where persona "hurts" and outcome-first beats "act as expert". On Gemini 3, **keep persona blocks** — they're load-bearing.

⚠️ **Cross-vendor compromise**: a prompt with persona is GOOD for Gemini, NEUTRAL for Claude, BAD for GPT-5.5. The `/prompt-atlas` skill must detect this contradiction and ask the user which target(s) before recommending strip-or-keep on persona blocks.

Place identity prompts in **System Instructions** or at the **very top** of the prompt — Google explicitly: *"Place behavioral constraints and role definitions in the System Instruction or at the very top of the prompt to ensure they anchor the model's reasoning process."*

### 5. Persona handling — strong adherence (gotcha)

> *"The model treats assigned personas seriously and will sometimes ignore instructions in order to maintain adherence to the described persona."*

This is a unique Gemini gotcha. Persona is high-leverage but also high-risk: an over-specified persona can override later task instructions.

When reviewing a Gemini prompt with both a strong persona AND specific task constraints:
- Verify persona is consistent with task (e.g., persona "creative writer" + task "produce JSON output" → persona will fight format)
- If conflict: either soften persona or move task constraints into System Instructions where persona lives

### 6. Negative constraints go AT THE END

> *"The model may **drop negative constraints** if they appear too early in the prompt. Place negative, formatting, and quantitative constraints at the end."*

Suggested order for complex Gemini 3 prompts:
1. Context and source material (top)
2. Main task instructions
3. Negative / formatting / quantitative constraints (bottom)

This is **stricter** than Claude/GPT-5.x where constraint position is more forgiving.

### 7. Anti-pattern — blanket "do not" instructions

Official guidance: *"Open-ended system instructions like 'do not infer' or 'do not guess' may cause the model to over-index on that instruction and fail to perform basic logic or arithmetic. Rather than a large blanket negative constraint, tell the model explicitly to use the provided context for deductions and avoid using outside knowledge."*

Replace blanket negatives with positive scoped instructions:

| ❌ Blanket negative | ✅ Scoped positive |
|---|---|
| "Do not infer." | "Use the provided documents for deductions. If the answer isn't in them, say 'I don't know'." |
| "Do not guess." | "If a value isn't explicitly given in the data, return `null`." |
| "Do not hallucinate." | "Cite the document section for every factual claim." |

### 8. XML or Markdown — pick ONE, don't mix

> *"Employ clear delimiters to separate different parts of your prompt, using XML-style tags (e.g., `<context>`, `<task>`) or Markdown headings. **Choose one format and use it consistently within a single prompt.**"*

Mixing breaks the model's parsing of prompt structure. When reviewing a Gemini prompt with both XML tags and `## Markdown headings` → `[IMPROVE]` finding: pick one consistently.

This is stricter than Claude/GPT-5.x where mixing is tolerated (though not optimal).

### 9. Long-context anchoring

> *"When working with large datasets (e.g., entire books, codebases, or long videos), place your specific instructions or questions at the **end** of the prompt, after the data context. Anchor the model's reasoning to the provided data by starting your question with a phrase like 'Based on the information above...'"*

Same pattern as Claude and GPT-5.x. The "Based on the information above" anchor is Google's recommended phrasing.

### 10. Multimodal — text/image/audio/video equal-class

Gemini 3 treats multimodal inputs as first-class peers. When reviewing a Gemini prompt that has images/audio/video alongside text:
- Don't write "I've attached an image" — it's redundant; the model sees it
- Reference each modality by its position or name, not by introducing it
- Same XML/Markdown structure rule applies — wrap each input in a tag

### 11. Default tone — direct, not conversational

> *"By default, Gemini 3 is less verbose and prefers providing direct, efficient answers. If conversational output is needed, you must explicitly steer the model in the prompt (e.g., 'Explain this as a friendly, talkative assistant')."*

Same as GPT-5.5's default. Cross-vendor: tone-prompts that worked on Gemini 2.5 ("be friendly", "use casual language") may need to be more explicit on 3.

### 12. Tool use — function calling + thought signatures

For agentic Gemini 3 prompts using tools:

- **Combining built-in tools (Google Search, Maps, Code Execution, URL Context, File Search) with custom function calling is now supported.**
- **Thought signatures** are required to maintain reasoning context across turns. *"The API enforces strict validation on the 'Current Turn'. Missing signatures will result in a 400 error."*

This is configuration, not wording — but flag in reviews of Gemini agentic prompts: if the tool-use loop is breaking, check signatures before rewriting body.

### 13. Output format — `json_schema` preferred

Same as GPT-5.x: push output contracts into API parameters rather than prose. Use `response_mime_type: "application/json"` + `response_json_schema: {...}` rather than describing JSON shape in the system prompt.

### 14. Image segmentation removed in Gemini 3

If the prompt relies on image segmentation features → image segmentation is **no longer supported in Gemini 3 Pro/Flash**. Use Gemini 2.5 Flash or Robotics-ER 1.6 instead. This is a capability change worth flagging in migration reviews.

---

## Gemini 3.1 Pro (March 2026)

The frontier model, optimized for deep reasoning and long-context work.

### Behaviors that shape wording

- **`thinking_level: high` is the default** — internal reasoning is generous by default. Explicit "think harder" wording is dead weight.
- **1M+ context window** — long-document techniques apply (data top, question bottom, "Based on the above..." anchor)
- **Strongest persona adherence** — gotcha #5 most pronounced here
- **Identity-based +5% boost** most measurable on complex reasoning tasks

### Migration from Gemini 3 Pro Preview (deprecated 2026-03-09)

Direct replacement: `gemini-3.1-pro-preview`. Wording is unchanged from 3.0 → 3.1; mostly an SDK-side migration. Audit:
- `google-generativeai` package (EOL Nov 2025) → migrate to `google-genai`
- Legacy `thinking_budget` → new `thinking_level`
- `media_resolution_high` setting if PDFs / dense documents are in use

---

## Gemini 3 Flash

Balanced model — speed and capability between Pro and Flash-Lite.

### Behaviors that shape wording

- **Same family rules apply** — concise, identity-based, no temperature tuning, no CoT
- **`thinking_level: low` is a common choice for latency-sensitive use** — if so, system instruction *"think silently"* helps reduce visible-reasoning overhead
- **Few-shot tolerance is more forgiving than Pro** — when format is genuinely strict, examples help here more than on Pro

### When to use Flash vs Pro

Flash is the default for interactive sessions, agentic loops, and high-volume work. Pro is for one-shot deep reasoning, long-context analysis, hard problems. Wording is the same; reasoning depth differs.

---

## Gemini 3.5 Flash (May 19, 2026 — current Flash frontier)

A larger jump than the version number suggests. Beats Gemini 3.1 Pro on **coding and agentic benchmarks** (Terminal-Bench 2.1 76.2%, MCP Atlas 83.6%, GDPval-AA 1656 Elo) while 3.1 Pro still wins **abstract reasoning** (Humanity's Last Exam +4.2 pts, ARC-AGI-2 +5).

### Critical migration gotcha — `thinking_level` default dropped

The integer `thinking_budget` from Gemini 3 Flash Preview is **retired**. New `thinking_level` enum: `minimal` / `low` / `medium` (**default**) / `high`.

**Silent regression risk:** the default moved from `high` (on the Preview) to `medium`. A naive port `gemini-3-flash-preview` → `gemini-3.5-flash` reasons less than the old prompt did, with no error. Audit prompts that depended on the prior default; set `thinking_level: high` explicitly if the workload needs it.

**Also retuned:** `low` is no longer "skip reasoning" — Google retuned it specifically for code/agentic tasks. Use `low` deliberately for agentic loops where you want fast turns; don't avoid it as if it disables thinking.

### Wording differences from 3 Flash

- Family rules unchanged (concise, identity-based, no temperature tuning, no CoT scaffolding)
- Coding benchmarks now Flash-leading — older Flash prompts that punted hard coding to Pro can be revisited
- Long-context (1M) behavior same; documents-top / question-bottom anchor still applies

### When 3.5 Flash specific tuning helps

- Agentic coding loops migrating from 3.1 Pro for cost/speed
- High-volume agentic workloads where 3.5 Flash now matches Pro on coding-shaped tasks
- Migration audits from any `gemini-3-flash-preview` codebase

---

## Gemini 3.1 Flash-Lite (March 2026)

Speed/cost-optimized — 363 tokens/sec, $0.25/$1.50 per 1M tokens, 1M context. Designed for high-volume, latency-sensitive workloads.

### Behaviors that shape wording

- **Concise wording is critical** — every token costs latency at this throughput
- **`thinking_level: low` is typical** — explicit "think silently" in system instruction is common
- **Few-shot for format helps more than on Pro** — Lite needs more guidance for strict formats
- **Bounded scope wins** — like Haiku 4.5, Lite excels at classification, extraction, summarization, translation. If you find yourself writing a 10-step workflow for Lite, promote to Flash or Pro
- **Tool descriptions matter more** — push tool guidance into descriptions, keep system prompt minimal

### As a subagent

Lite is the right default when the subagent's job is bounded and read-only: classifiers, extractors, cheap pre-filters, language tasks. Use Flash or Pro for the main conversation where reasoning quality matters.

---

## Gemini 2.5 Pro / Flash (legacy — for migration reviews)

Older generation. When migrating from 2.5 to 3.x, the changes are large enough that Google recommends a fresh-baseline approach:

### Wording fixes when migrating 2.5 → 3.x

| Old 2.5-style | 3.x-style |
|---|---|
| "Think step by step." / "First, do X. Then do Y." | (Delete.) Set `thinking_level: high` if reasoning is shallow. |
| Long block of "DO NOT do X / Y / Z" | Replace with positive scoped instructions ("Use the provided context for deductions"). Move remaining negatives to the END of the prompt. |
| Few-shot block of 5 examples | Test zero-shot first; keep examples only if format drifts. |
| Mixed XML + Markdown structure | Pick one. |
| Persona absent or weak | **Add identity-based persona** ("You are a planner...") — measurable boost. |
| `temperature: 0.3` for deterministic output | **Remove.** Gemini 3 default 1.0 is required. |
| `thinking_budget: <number>` | Migrate to `thinking_level: minimal/low/medium/high`. |

---

## Gemini-specific anti-patterns

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **Chain-of-thought scaffolding** | "Think step by step" + numbered mental moves in the body | Strip; raise `thinking_level` to medium or high |
| **Temperature in prompt body** | "Set temperature to 0.3" mentioned in body | Remove from body; the API param itself should stay at 1.0 default |
| **Mixed XML + Markdown** | `<context>...</context>` and `## Section` in same prompt | Pick one structure; prefer XML for inline, Markdown for top-level structure |
| **Blanket "do not"** | `"do not infer / do not guess / do not hallucinate"` | Replace with positive scoped instructions |
| **Negative constraints at top** | "Don't include X" in the first paragraph | Move to end; model drops early negatives |
| **Missing persona** | Prompt has no identity-based system instruction | Add 1 line: "You are a [planner / analyst / writer]" — measurable boost |
| **Over-strong persona vs task** | Strong persona conflicts with output format requirements | Soften persona OR move format constraints into System Instructions where persona lives |
| **Hardcoded model name** | "You are Gemini 3.1 Pro" | Strip; use functional role |
| **Legacy `thinking_budget`** | Old SDK pattern in code | Migrate to `thinking_level` (cannot coexist — 400 error) |

---

## Cross-vendor implications

When the prompt is **cross-vendor** (Claude + GPT-5.x + Gemini), several Gemini-specific defaults conflict with other vendors:

| Axis | Gemini 3 | Claude | GPT-5.5 | Cross-vendor compromise |
|---|---|---|---|---|
| Persona / "act as" | **+5% boost — keep** | Neutral | Hurts | **Conditional persona** OR drop with -5% Gemini cost |
| Temperature in body | Don't tune | Tunable | Tunable | Don't reference temperature in body; let API config handle |
| Negative constraint position | At end (strict) | Anywhere | Anywhere | Place at end (strictest wins) |
| Structure mixing | XML or Markdown, not both (strict) | Tolerates mixing | Tolerates mixing | Pick one (strictest wins) |
| CoT scaffolding | Hurts | Tolerated | Hurts | Strip — works for all three |
| Blanket "do not" | Over-indexes | Tolerated | Tolerated | Replace with positive scoped instructions (works for all three) |

The persona conflict is the most painful: there's no neutral position. Cross-vendor `AGENTS.md` must either:
- **Drop persona entirely** (lose 5% on Gemini, gain on GPT-5.5)
- **Keep functional persona that's also a methodological anchor** ("Apply systems-thinking lenses, evaluate risk before recommending") — this works on all three vendors because it names *frames*, not *credentials*. See `gpt.md` and `claude.md` on methodological anchors.
- **Conditional persona** — split prompts by vendor (heavier maintenance but optimal)

When this conflict shows up in a review, the `/prompt-atlas` skill must **flag and ask** which target(s) the user actually has — see SKILL.md § "Contradiction detection".

---

## Universal Gemini 3.x prompts

A prompt that works across Pro / Flash / Flash-Lite:

- [ ] Concise (Flash-Lite needs it most)
- [ ] No CoT scaffolding (all three internalize reasoning)
- [ ] No temperature in body (all three default 1.0)
- [ ] Identity-based persona present (helps all three)
- [ ] XML or Markdown, not both
- [ ] Negative constraints at end
- [ ] Output format via `response_json_schema` if structured
- [ ] Tool guidance in tool descriptions, not system prompt
- [ ] No model name hardcoded
- [ ] If long context: data at top, question at end, "Based on the above..." anchor
