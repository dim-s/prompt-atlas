# Model-specific wording — Google Gemini (4 Argon, 3.x)

What changes about how you should PHRASE prompts for Gemini 4 Argon and Gemini 3.x. Companion to `claude.md` (Claude) and `gpt.md` (GPT). Focused on wording — not API parameter values, pricing, or context limits beyond what affects the prompt.

When the artifact runs in Gemini CLI specifically (GEMINI.md, AGENTS.md via fileName config, Gemini CLI subagent, skill, slash command), also read `../agentic-systems/gemini-cli.md` — the CLI layers its own behavior on top of the underlying model.

Retired, deprecated and older models (3 Flash, 3.5 Flash, 3.7 Flash, 3.1 Flash-Lite, 2.5) live in `../archive/models/gemini.md`.

---

## Family-wide rules (Gemini 3 guide; what holds on Gemini 4 Argon)

Rules 1–14 come from Google's prompt-design page, whose model-specific section is still titled "Gemini 3" (last updated 2026-10-13), and from the Gemini 3 developer guide. **Google has published no Gemini 4 prompting guide.** Argon's migration page (`/gemini-api/docs/latest-model`) is API-only: it neither confirms nor retracts any wording rule. So the rules hold on 3.x as documented; on Argon, the wording rules are the best available baseline, not Argon-documented facts — carry them over and test.

| # | Rule | Gemini 3.x | Gemini 4 Argon |
|---|---|---|---|
| 1 | Concise prompts | documented | not restated — carry over, unverified |
| 2 | No CoT scaffolding; depth via `thinking_level` | documented | lever documented (`low` / `medium` / `high`, default `medium`, no `minimal`); the CoT warning is not restated |
| 3 | No sampling tuning | 3.6+: values ignored; earlier 3.x: keep defaults | documented: values ignored, remove them |
| 4–11 | Identity persona, persona adherence, negatives at the end, no blanket "do not", one structure format, long-context anchoring, multimodal as peers, direct default tone | documented | not restated — carry over, unverified |
| 12 | Thought signatures across turns | documented | generic thinking page applies; migration page: keep state on the server (`previous_interaction_id`) |
| 13 | Output contract in `response_json_schema` | documented | structured outputs "Supported" (model page) |
| 14 | No image segmentation | documented for 3 Pro / 3 Flash | `?` — the Argon model page doesn't list it |

### 1. Reasoning model — concise prompts win

Google's official prompting guide is explicit: *"Gemini 3 is a reasoning model. Be concise in your input prompts. Gemini 3 responds best to direct, clear instructions and may **over-analyze** verbose or overly complex prompt engineering techniques used for older models."*

Practical impact: prompt-stacks ported from Gemini 2.5 (or worse, from older GPT/Claude) with extensive scaffolding will **underperform** versus a leaner zero-baseline rewrite.

Frontier reasoning models across vendors share this preference for concise prompts. The patches accumulated against earlier-version quirks now address problems the new model doesn't have.

### 2. Chain-of-thought scaffolding becomes harmful

Google's migration guide: *"If you were previously using complex prompt engineering (like chain of thought) to force Gemini 2.5 to reason, try Gemini 3 with `thinking_level: 'high'` and simplified prompts."*

When porting a 2.5-tuned prompt: **strip CoT scaffolding entirely**. Don't leave it "just in case" — internal thinking happens automatically on Gemini 3, and explicit CoT competes with it.

When reviewing a Gemini prompt that says *"think step by step"*, *"first do X, then Y, then Z"*, *"break this down"* → flag as `[IMPROVE]` and recommend deletion + raise `thinking_level` if reasoning is shallow. On Argon the same lever exists, and Google's advice is to pick it by eval, not by prose: *"Run your evaluations at all three levels, then pick the lowest level that meets your quality bar."*

### 3. Sampling parameters — ignored on Gemini 4 and 3.6+ (was: don't tune)

Google's long-standing recommendation was to leave temperature alone:

> *"For Gemini 3, we strongly recommend keeping the temperature parameter at its default value of 1.0. Changing the temperature (setting it below 1.0) may lead to unexpected behavior, looping, or degraded performance, particularly with complex mathematical or reasoning tasks."*

On 2026-07-21 the changelog made it a deprecation: *"The sampling parameters temperature, top_p and top_k are now deprecated."* The current prompt-design page states the split: *"Gemini 4 and Gemini 3.6+ models ignore custom values for `temperature`, `top_p`, and `top_k` and return deprecation warning headers (and will return `400` errors in a future release)… For earlier Gemini 3 models, keep these parameters at their default values to avoid degraded reasoning performance."* Argon's migration checklist: *"Remove `temperature`, `top_p`, and `top_k` from your requests. If provided, they are ignored."* Steering tone, determinism and variety through sampling is no longer an option on this vendor — only through wording.

When reviewing a Gemini prompt:
- If the prompt body mentions temperature → flag for removal (don't reference API params in prompt body anyway)
- If the surrounding code sets `temperature`, `top_p` or `top_k` → flag as a non-wording finding: on Argon / 3.6+ the values already do nothing, and a future release turns them into 400s
- For variety across runs (design work, naming, creative options), use the **propose-N-directions** pattern instead of sampling — the same replacement the sampling-locked Claude models needed
- For cross-vendor `AGENTS.md`: don't reference sampling parameters at all; let each vendor's API config handle what it still supports

**This is a cross-vendor trend, not a Gemini quirk.** Anthropic rejects the three parameters on every Claude model since API version `2026-10-21` (`claude.md` § Cross-model wording principles), Google deprecated them in July 2026, and OpenAI's GPT-6 Astra lists `temperature` / `top_p` as unsupported. Which models still tune is tracked in `_universal.md § Cross-vendor`. The safe universal rule is the strict one: **treat sampling as unavailable and steer with words.**

### 4. Identity-based prompting — `+5%` reasoning boost

Google's research: *"Identity-based prompting — telling the model what kind of thinker it is — boosted performance by around 5% on complex reasoning tasks."*

Effective patterns:
- *"You are a planner."*
- *"You must analyze before responding."*
- *"You should evaluate risk and reorder steps."*

**This is the OPPOSITE default to GPT**, where outcome-first beats "act as expert" (`gpt.md` family rule #12). On Gemini 3, **keep persona blocks** — they're load-bearing.

⚠️ **Cross-vendor compromise**: a prompt with persona is GOOD for Gemini, NEUTRAL for Claude, BAD for GPT. The `/prompt-atlas` skill must detect this contradiction and ask the user which target(s) before recommending strip-or-keep on persona blocks.

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

This is **stricter** than Claude/GPT where constraint position is more forgiving.

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

This is stricter than Claude/GPT where mixing is tolerated (though not optimal).

### 9. Long-context anchoring

> *"When working with large datasets (e.g., entire books, codebases, or long videos), place your specific instructions or questions at the **end** of the prompt, after the data context. Anchor the model's reasoning to the provided data by starting your question with a phrase like 'Based on the information above...'"*

Same pattern as Claude and GPT. The "Based on the information above" anchor is Google's recommended phrasing.

### 10. Multimodal — text/image/audio/video equal-class

Gemini 3 treats multimodal inputs as first-class peers. When reviewing a Gemini prompt that has images/audio/video alongside text:
- Don't write "I've attached an image" — it's redundant; the model sees it
- Reference each modality by its position or name, not by introducing it
- Same XML/Markdown structure rule applies — wrap each input in a tag

### 11. Default tone — direct, not conversational

> *"By default, Gemini 3 is less verbose and prefers providing direct, efficient answers. If conversational output is needed, you must explicitly steer the model in the prompt (e.g., 'Explain this as a friendly, talkative assistant')."*

Same direction as GPT's default. Cross-vendor: tone-prompts that worked on Gemini 2.5 ("be friendly", "use casual language") may need to be more explicit on 3.

### 12. Tool use — function calling + thought signatures

For agentic Gemini 3 prompts using tools:

- **Combining built-in tools (Google Search, Maps, Code Execution, URL Context, File Search) with custom function calling is now supported.**
- **Thought signatures** are required to maintain reasoning context across turns. *"The API enforces strict validation on the 'Current Turn'. Missing signatures will result in a 400 error."* In the Interactions API's stateful mode (`previous_interaction_id`) the server manages them; Argon's migration checklist points multi-turn and function-calling conversations there.

This is configuration, not wording — but flag in reviews of Gemini agentic prompts: if the tool-use loop is breaking, check signatures before rewriting body.

### 13. Output format — `json_schema` preferred

Same as GPT: push output contracts into API parameters rather than prose. Use `response_mime_type: "application/json"` + `response_json_schema: {...}` rather than describing JSON shape in the system prompt.

### 14. Image segmentation removed in Gemini 3

If the prompt relies on image segmentation → Google's Gemini 3 guide: *"Image segmentation capabilities (returning pixel-level masks for objects) are not supported in Gemini 3 Pro or Gemini 3 Flash. For workloads requiring built-in image segmentation, we recommend continuing to use Gemini 2.5 Flash with thinking turned off."* That fallback is now narrow: since 2026-09-18 Gemini 2.5 access is limited *"to users who have actively used them in the past"*, and Robotics-ER 1.6 is shut down. For a new project no current Gemini model is documented to do segmentation (Argon's model page doesn't list it) — flag it as a capability gap in migration reviews, not a wording problem.

---

## Gemini 4 Argon (October 13, 2026 — current Pro tier, first Gemini 4)

`gemini-4-argon`, GA on release (stable id, not on the Free tier). Google's *"flagship reasoning model"*, above 3.1 Pro, and the first Gemini model outside 3.x. No other Gemini 4 model exists yet — the launch post: *"We're looking forward to bringing you the rest of the Gemini 4 family."*

**No prompting guide.** The baseline is the family rules above, with the Argon status column as the honest caveat. What follows is the documented API-side change set, plus the launch post's behavior claims kept separate.

### Documented changes that shape wording

- **Default depth drops from 3.1 Pro's `high` to `medium`** (levels `low` / `medium` / `high`; `minimal` is not listed). A prompt ported from 3.1 Pro reasons less by default with no error — the same silent-regression class as the archived 3.5 Flash default drop. The fix is `thinking_level: high` where the workload needs it, not "think harder" prose. Google's level guide: `medium` *"Recommended for most tasks, including complex code and agentic use cases"*; `high` for *"Complex reasoning, hard math and science, and the most difficult code or agent tasks."*
- **Thinking eats the output budget.** *"Gemini 4 Argon can spend more of the limit on thinking than earlier models. To use fewer tokens, lower the thinking level instead."* So a "keep your reasoning short" or "be brief" line is not the lever, and a small `max_output_tokens` truncates answers (Google: raise it to at least 32,768).
- **One 1M window for input, thinking and output** (*"Input, thinking, and output share one window of 1,048,576 tokens"*). A prompt architecture that fills ~1M with input leaves no room to answer. A single answer over 64K needs background execution (long decode) — a prompt asking for a book-length single output fails synchronously.
- **Sampling ignored** — family rule #3. Variety comes from wording.
- **Explicit caching unsupported; implicit caching only** (on by default for prompts of at least 4,096 tokens). Derived consequence: keep the stable part of the prompt (system instruction, tool declarations, reference docs) as an unchanging prefix and put per-call content after it — there is no explicit cache handle to compensate for a reshuffled prefix.
- **Computer use supported** (GA across the Interactions and GenerateContent APIs on 2026-10-30). Live API not supported.

### Launch-post behavior claims — marketing, not guidance

The launch post (blog.google, 2026-10-13) says: *"It communicates clearly, keeping you in the loop, and follows style instructions better than any of our previous Gemini models. Argon answers questions more directly with less hedging, doesn't repeat itself more than necessary, and skips clichés throughout the responses."* And: *"we trained Argon to respect system boundaries, report negative findings honestly, and pause for human confirmation before taking irreversible steps."* And on injection: *"Argon leads the Gray Swan Indirect Prompt Injection (IPI) benchmark, cutting attack success across 15 adaptive attempts to 0.7%."*

These are vendor claims with no prompting guidance attached. Do not turn them into rules. In reviews:
- Anti-hedging, anti-repetition and anti-cliché lines carried from 3.x *may* be redundant on Argon — that is a hypothesis to test on the user's own eval, not a documented delta; keep them until an eval says otherwise.
- Keep explicit confirmation rules for irreversible actions and "treat retrieved content as data" instructions in agent prompts — a trained tendency is not a guarantee, and the prompt is where the operator's boundary is written down.

### Migrating a prompt from 3.1 Pro

- Wording: carry over as-is — Google says nothing either way about wording changes.
- Set `thinking_level` explicitly if the 3.1 Pro prompt relied on the `high` default.
- Remove sampling params and explicit-cache calls from the surrounding code (non-wording findings).
- Raise `max_output_tokens` to at least 32,768 if it was set lower.

---

## Gemini 3.1 Pro (February 2026 preview — previous Pro tier)

`gemini-3.1-pro-preview`, released 2026-02-19 and still a preview. Superseded at the top by Gemini 4 Argon on 2026-10-13; still served, no shutdown date announced. Optimized for deep reasoning and long-context work.

### Behaviors that shape wording

- **`thinking_level: high` is the default** (levels `low` / `medium` / `high`) — internal reasoning is generous by default. Explicit "think harder" wording is dead weight.
- **1M+ context window** — long-document techniques apply (data top, question bottom, "Based on the above..." anchor)
- **Strongest persona adherence** — gotcha #5 most pronounced here
- **Identity-based +5% boost** most measurable on complex reasoning tasks
- **Sampling: keep defaults** — as an earlier Gemini 3 model it is not in the "ignored" set (family rule #3)

### Migration from Gemini 3 Pro Preview (shut down 2026-03-09)

Direct replacement: `gemini-3.1-pro-preview`. Wording is unchanged from 3.0 → 3.1; mostly an SDK-side migration. Audit:
- `google-generativeai` package (EOL Nov 2025) → migrate to `google-genai`
- Legacy `thinking_budget` → new `thinking_level`
- `media_resolution_high` setting if PDFs / dense documents are in use

---

## Gemini 3.8 Flash (September 2, 2026 — current Flash frontier)

`gemini-3.8-flash`, GA on release, three weeks after 3.7 Flash — Google's post itself notes it is *"marking our third Flash release in only six weeks"*. Positioned as *"our most intelligent workhorse model, delivering significant improvements from 3.7 Flash across software engineering, agentic tasks, and critical, multi-step reasoning in specialized domains."*

**A prompt pinned to `gemini-3.7-flash` now runs on 3.8 Flash.** Google deprecated 3.7 Flash on 2026-10-08: *"API requests to gemini-3.7-flash are automatically routed to gemini-3.8-flash."* No error, no shutdown — the prompt silently gets the harder-working model below. Review it as a 3.8 Flash prompt: strip persistence scaffolding, re-check token budgets, and update the model string.

### Behaviors that shape wording

- **It works harder by design — the Flash terseness trend reverses.** Google: *"On complex tasks, it exhibits greater diligence — executing extra reasoning steps, and calling tools iteratively. At times, the model might use more tokens to maximize performance, especially at higher effort levels."* 3.6 Flash was ~17% terser than 3.5 and the atlas flagged carried-over "be brief" lines as overcorrecting; 3.8 moves the other way. Derived consequence: don't port persistence or thoroughness scaffolding ("keep calling tools until you're sure", "be exhaustive", "double-check with another search") into a 3.8 prompt — it stacks on a model that already takes the extra steps.
- **Efficiency is a parameter or model choice, not prose.** The launch post advised *"developers can utilize lower effort levels to minimize token overhead or continue to rely on Gemini 3.7 Flash, which remains fully supported for efficiency-first workloads."* The second half no longer holds — 3.7 Flash now routes to 3.8 Flash. Lower effort is the remaining lever; a "be brief" line won't recover tokens the model spends by design.
- **`thinking_level` `low` / `medium` / `high`, default `medium`; `minimal` returns an error** (API model page and thinking docs) — the same ladder as Argon. Configs carried from 3.5-era `minimal` settings fail; the fix is `low`, not a "think silently" line.
- **Prompt-injection robustness** — *"Gemini 3.8 models have also made a significant leap in prompt injection robustness as measured by Gray Swan"*. Helpful for agents reading untrusted content; not a reason to remove "treat retrieved content as data" instructions.
- Family rules unchanged: concise input, identity-based persona, no CoT scaffolding, negatives at the end, one structure format, sampling values ignored.

### When 3.8 Flash specific tuning helps

- Long-horizon coding agents and professional-analysis agents (Google cites Vals Finance Agent V2, Harvey's Legal Agent Benchmark, 54.9% on HLE-Verified) — where the extra diligence is the point.

### When NOT to invest

- Efficiency-first, high-volume loops — lower effort (or 3.6 Flash / 3.5 Flash-Lite) is the fix; rewording won't undo tokens the model spends by design.

**No model-specific prompting guide was published for 3.8 Flash.** Everything above is quoted from or derived from Google's launch post (2026-09-02), the Gemini API changelog entry of the same date, the `gemini-3.8-flash` model page and the thinking docs (read 2026-09-13; routing re-read 2026-10-31).

---

## Gemini 3.8 Flash Cyber (September 2, 2026 — closed access)

A cybersecurity variant built on the same core — both releases *"are powered by the same foundational intelligence"* — and, like the earlier 3.5 Flash Cyber, **not generally available**: *"available to trusted defenders through our new Fairwind Program"* (government authorities, critical infrastructure operators, software maintainers). Note the access route changed from 3.5 Flash Cyber's CodeMender. Nothing to tune against; recorded so a later pass doesn't re-investigate.

One consequence for the GA model: *"3.8 Flash ships with safeguards against misuse in the domains of Chemical, Biological, Radiological, and Nuclear (CBRN) and cyber offense"* — the permissive cyber mitigations belong to the Cyber variant only. No wording workaround is documented for benign security prompts that hit those safeguards on 3.8 Flash.

---

## Gemini 3.6 Flash (July 21, 2026 — previous Flash frontier)

`gemini-3.6-flash`, GA on release; the models page calls it *"Our previous-generation Flash model"*. The changelog describes it as offering *"improved token efficiency and code/agentic planning capabilities at a lower price point than 3.5 Flash."*

**A prompt pinned to `gemini-3.5-flash` now runs on 3.6 Flash** — 3.5 Flash was deprecated on 2026-10-08 and its requests are *"automatically routed to gemini-3.6-flash"*. Such a prompt silently gets the terser model below; re-check its length instructions.

### Behaviors that shape wording

- **Output is ~17% shorter than 3.5 Flash** at equal or better quality (up to 65% fewer tokens on DeepSWE-class agentic coding). Like GPT-5.6, this inverts carried-over concision instructions: a "be brief / no preamble" line written against 3.5 Flash now stacks on an already-terser model. Express length as a task requirement, not as a disposition.
- **Fewer unwanted edits and fewer execution loops** — Google reports higher precision on agentic coding (DeepSWE 49% vs 37%). Scaffolding added to stop 3.5 Flash from thrashing (retry rules, "don't edit files you weren't asked to" walls) is a candidate for trimming — re-test before keeping it.
- **`thinking_level` default `medium`; `minimal` still accepted** (levels `minimal` / `low` / `medium` / `high`) — unlike 3.8 Flash and Argon.
- **Family rules unchanged**: concise input, identity-based persona, no CoT scaffolding, negatives at the end, one structure format.
- **Sampling values ignored** — deprecated on its release date; see family rule #3.

### When 3.6 Flash specific tuning helps

- Cost/latency-sensitive agentic loops migrating off 3.5 Flash — re-check any length instructions and loop-damping scaffolding while you're there
- High-volume workloads where a 17% output reduction compounds

**No model-specific prompting guide was published for 3.6 Flash** — Google documented the model, not new prompting advice. Everything above is derived from documented behavior; the wording rules remain the family ones.

---

## Gemini 3.5 Flash-Lite (July 21, 2026 — the vendor's designated subagent model)

`gemini-3.5-flash-lite`, GA on release. Google's changelog positions it explicitly: *"a low-latency, highly cost-effective subagent option designed for high-volume automation."* This is the first time a vendor in this atlas ships a model **named for the subagent role** rather than merely suited to it. It is also the designated replacement for 3.1 Flash-Lite (earliest shutdown 2027-05-07; section in `../archive/models/gemini.md`).

### Behaviors that shape wording

- The Lite regime: concise wording (every token costs latency), bounded scope, few-shot for strict formats helps more than on Pro, tool guidance pushed into tool descriptions with a minimal system prompt.
- **`thinking_level` default `minimal`** (levels `minimal` … `high`).
- **Stronger than the previous Lite tier at agentic work** — SWE-Bench Pro 54.2% vs 49.6% for 3 Flash, at ~350 output tokens/sec. Tasks previously escalated from Lite to Flash on capability grounds are worth re-testing.

### As a subagent

When a Gemini-side prompt review involves a subagent fleet, this is the default worker: bounded, read-only or narrowly-scoped jobs (search, classification, extraction, pre-filtering), with the orchestrator on Argon, 3.1 Pro or Flash. The wording implications are the standard ones for a cheap executor — spell out the success criteria, name the tools, keep the scope small, and put the format contract in `response_json_schema` rather than prose.

The mirror-image models across vendors are the cheap volume tiers (Claude Haiku, GPT Luna — see `claude.md` / `gpt.md`). A subagent prompt written for one usually ports to the others with only the format-contract syntax changing.

---

## Gemini-specific anti-patterns

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **Chain-of-thought scaffolding** | "Think step by step" + numbered mental moves in the body | Strip; raise `thinking_level` to medium or high |
| **Sampling params in prompt body** | "Set temperature to 0.3" / `top_p` / `top_k` mentioned in body | Remove from body — and the params themselves are **ignored on Argon / 3.6+** (400 in a future release), so the code around it needs a fix too. For variety, use propose-N-directions |
| **Mixed XML + Markdown** | `<context>...</context>` and `## Section` in same prompt | Pick one structure; prefer XML for inline, Markdown for top-level structure |
| **Blanket "do not"** | `"do not infer / do not guess / do not hallucinate"` | Replace with positive scoped instructions |
| **Negative constraints at top** | "Don't include X" in the first paragraph | Move to end; model drops early negatives |
| **Missing persona** | Prompt has no identity-based system instruction | Add 1 line: "You are a [planner / analyst / writer]" — measurable boost on 3.x (unverified on Argon) |
| **Over-strong persona vs task** | Strong persona conflicts with output format requirements | Soften persona OR move format constraints into System Instructions where persona lives |
| **Hardcoded model name** | "You are Gemini 3.1 Pro" | Strip; use functional role |
| **Legacy `thinking_budget`** | Old SDK pattern in code | Migrate to `thinking_level` (cannot coexist — 400 error) |
| **"Be brief" to save thinking tokens** | Prose asks the model to think less | Lower `thinking_level` — Google's stated lever on Argon |

---

## Cross-vendor implications

When the prompt is **cross-vendor** (Claude + GPT + Gemini), several Gemini-specific defaults conflict with other vendors:

| Axis | Gemini 3.x (Argon: carried over, unverified) | Claude | GPT | Cross-vendor compromise |
|---|---|---|---|---|
| Persona / "act as" | **+5% boost — keep** | Neutral | Hurts | **Conditional persona** OR drop with -5% Gemini cost |
| Sampling params | **ignored on Argon / 3.6+**; earlier 3.x: defaults | → 400 on every model (API version `2026-10-21`) | Tunable on 5.x; GPT-6 Astra: unsupported | Don't reference sampling in body, and don't design around it |
| Response length default | 3.6 Flash ~17% terser than 3.5; 3.8 Flash works harder; Argon `?` | **Opus 5 runs long, doesn't calibrate** | 5.6 terser than 5.5 | State length as a task requirement, not a disposition — the defaults move in opposite directions per model |
| Negative constraint position | At end (strict) | Anywhere | Anywhere | Place at end (strictest wins) |
| Structure mixing | XML or Markdown, not both (strict) | Tolerates mixing | Tolerates mixing | Pick one (strictest wins) |
| CoT scaffolding | Hurts | Tolerated | Hurts | Strip — works for all three |
| Blanket "do not" | Over-indexes | Tolerated | Tolerated | Replace with positive scoped instructions (works for all three) |

The persona conflict is the most painful: there's no neutral position. Cross-vendor `AGENTS.md` must either:
- **Drop persona entirely** (lose 5% on Gemini, gain on GPT)
- **Keep functional persona that's also a methodological anchor** ("Apply systems-thinking lenses, evaluate risk before recommending") — this works on all three vendors because it names *frames*, not *credentials*. See `gpt.md` and `claude.md` on methodological anchors.
- **Conditional persona** — split prompts by vendor (heavier maintenance but optimal)

When this conflict shows up in a review, the `/prompt-atlas` skill must **flag and ask** which target(s) the user actually has — see SKILL.md Step 4 (contradiction detection).

---

## Universal Gemini prompts (Argon + 3.x)

A prompt that works across Argon / Pro / Flash / Flash-Lite:

- [ ] Concise (Flash-Lite needs it most)
- [ ] No CoT scaffolding (all internalize reasoning)
- [ ] No sampling params in body (ignored on Argon / 3.6+; keep defaults on earlier 3.x)
- [ ] Length stated as a task requirement, not a disposition (3.6 Flash is terser than 3.5, 3.8 Flash works harder — one global "be brief" is wrong for one of them)
- [ ] Identity-based persona present (documented for 3.x; carry to Argon and test)
- [ ] XML or Markdown, not both
- [ ] Negative constraints at end
- [ ] Output format via `response_json_schema` if structured
- [ ] Tool guidance in tool descriptions, not system prompt
- [ ] No model name hardcoded
- [ ] If long context: data at top, question at end, "Based on the above..." anchor
- [ ] Stable prefix first, per-call content last (Argon caches implicitly only)

---

## Source notes

Facts added or re-checked in v2.0.0 (2026-10-31):

- Argon id, GA, "flagship reasoning model", capabilities (structured outputs, computer use, Live API, implicit-only caching), 1M unified window, 64K synchronous / 1M long-decode output — https://ai.google.dev/gemini-api/docs/models/gemini-4-argon, read 2026-10-31 (raw; "Last updated 2026-10-30").
- Argon thinking levels and level guidance, "evaluate at all three levels", 32,768 output minimum and "spend more of the limit on thinking", "Input, thinking, and output share one window", sampling "ignored", explicit caching unsupported, implicit caching ≥4,096 tokens, `previous_interaction_id` — https://ai.google.dev/gemini-api/docs/latest-model (`.md.txt`), read 2026-10-31 (raw; "Last updated 2026-10-27"). The page has no prompting section.
- Thinking table (Argon / 3.8 Flash `low`–`high` default `medium`; 3.6 Flash default `medium` with `minimal`; 3.5 Flash-Lite default `minimal`; 3.1 Pro preview default `high`, `low`–`high`) and thought signatures in stateful mode — https://ai.google.dev/gemini-api/docs/thinking.md.txt, read 2026-10-31 (raw). The page's Go samples name a `gemini-3.8-pro`; no catalog, changelog or deprecation entry lists it — a doc artifact, not a model.
- Sampling split (Gemini 4 and 3.6+ ignore, 400 in a future release; earlier Gemini 3 keep defaults) and the absence of a Gemini 4 prompting section — https://ai.google.dev/gemini-api/docs/prompting-strategies, read 2026-10-31 (raw; "Last updated 2026-10-13").
- Image segmentation quote — https://ai.google.dev/gemini-api/docs/gemini-3 (Gemini 3 developer guide), read 2026-10-31 (raw).
- 3.7 Flash → 3.8 Flash and 3.5 Flash → 3.6 Flash routing (2026-10-08), Argon GA (2026-10-13), Computer Use GA (2026-10-30), 2.5 access limited to past users (2026-09-18), sampling deprecation (2026-07-21) — https://ai.google.dev/gemini-api/docs/changelog, read 2026-10-31 (raw).
- 3.1 Pro preview release 2026-02-19 / no shutdown date, 3 Pro preview shut down 2026-03-09, 3.1 Flash-Lite earliest shutdown 2027-05-07 → 3.5 Flash-Lite — https://ai.google.dev/gemini-api/docs/deprecations, read 2026-10-31 (raw; "Last updated 2026-10-22").
- 3.6 Flash "previous-generation Flash", Robotics ER 1.6 "Shut down", Gemini 4 section lists only Argon — https://ai.google.dev/gemini-api/docs/models, read 2026-10-31 (raw).
- Argon behavior claims and "rest of the Gemini 4 family" — https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-argon/, read 2026-10-31 (raw; vendor marketing, quoted, not used as guidance).
- Not read: the Gemini 4 Argon model card; the `whats-new-gemini-3.8` and `whats-new-gemini-3.6` pages; Vertex AI model pages.
