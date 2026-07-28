# Model-specific wording differences — OpenAI GPT-5.x

What actually changes about how you should PHRASE prompts for each current GPT-5.x model. Companion to `claude.md` (Claude). This reference stays focused on wording — not API parameter values, context sizes, or pricing.

When the artifact runs in OpenAI Codex CLI specifically (AGENTS.md, Codex subagent, Codex skill, Codex slash command), also read `../agentic-systems/codex.md` — Codex layers its own behavior on top of the underlying model and that interacts with prompt design.

---

## Family-wide rules (apply to all GPT-5.x versions)

These hold across 5.1 → 5.6. Version-specific notes follow below.

### 1. Outcome-first beats process-prescription

OpenAI's prompt-guidance is explicit: GPT-5.x performs best when you describe the **target outcome and success criteria**, not the steps to get there. From the official guide: *"GPT-5.5 treats detailed step-by-step instructions as interference: redundant instructions create noise, narrow the solution space, and make responses overly mechanical."* Earlier 5.x versions are less strict but trend the same way.

**On GPT-5.6 OpenAI put a number on it.** The latest-model guide reports that *"configurations with leaner system prompts improved evaluation scores by roughly 10–15% while reducing total tokens by 41–66% and cost by 33–67%."* That makes trimming the single highest-value edit on a 5.6-targeted review — not a stylistic preference. The guide's operational rule: remove repeated instructions, simplify tool descriptions, and state each instruction **once**.

| Weak (process-heavy) | Strong (outcome-first) |
|---|---|
| "First read the file, then identify the bug, then propose a fix, then write a test, then implement..." | "Fix the bug in `src/auth/session.py` that causes logout to fail after token refresh. Verification: the existing test suite passes plus a new regression test you add." |
| "Step 1: validate input. Step 2: query database. Step 3: format response." | "Endpoint must return 200 with the matching record on valid input, 404 on missing record, 400 on malformed input. Choose the implementation order yourself." |

This is the single biggest divergence from Claude defaults. Claude (especially Opus 4.7) often appreciates explicit step naming; GPT-5.5 reads that as noise.

### 2. Treat each new GPT-5.x version as a fresh baseline

OpenAI's GPT-5.5 migration guide is unusually direct: *"treat it as a new model family to tune for, not a drop-in replacement… begin migration with a fresh baseline instead of carrying over every instruction from an older prompt stack."*

Practical impact: a system prompt heavily tuned for GPT-5.4 — with patches accumulated against earlier versions' overengineering or hallucination quirks — often **underperforms** on GPT-5.5 versus a leaner zero-baseline rewrite. The accumulated patches each cost attention and now address problems the new model doesn't have.

When you see a long system prompt being ported across versions, flag this as a candidate for a from-scratch rewrite, not incremental tuning.

### 3. Tool-specific guidance lives in tool descriptions, not the system prompt

OpenAI's official guidance: *"Put most tool-specific guidance in the tool descriptions themselves: what the tool does, when to use it, required inputs, side effects, retry safety, and common error modes. Add tool-specific context to system instructions only when it applies across tools."*

This is a stronger split than Anthropic recommends. On GPT-5.x, system prompts crowded with per-tool usage hints (when to call X, what to pass to Y) are a sign the wording belongs inside the tool's `description` field instead.

### 4. Output contracts → API parameters, not prose

For structured outputs, prefer `json_schema` with `strict: true` over describing the JSON shape in prose. The system prompt should describe **behavior**, not **format**.

| Weak | Strong |
|---|---|
| "Return your answer as JSON with these fields: name (string), age (number), tags (array of strings). Make sure age is between 0 and 150. Do not add other fields." | (System prompt) "Extract person facts." (API call) `response_format: { type: "json_schema", json_schema: { schema: {...}, strict: true } }` |

If you're reviewing a prompt that fights the model into a specific JSON shape with bullet-list constraints in prose, the wording fix is "move this to `json_schema`". The skill is about wording, but pointing the user at the right knob is part of the review.

### 5. Few-shot examples are not free — and often hurt reasoning models

OpenAI explicitly warns: *"With today's reasoning models, clear instructions and well-defined constraints often work better than adding examples. Research shows few-shot prompts can reduce performance when the task requires heavy reasoning."*

Default to **zero-shot** with crisp constraints. Add examples only when:
- The output format is unusually strict and ambiguity is hard to remove with words alone, or
- You've tried zero-shot and observed format drift.

This is the opposite default from Claude, where 3–5 examples remain a good first move for steering format.

### 6. XML tagging for input structure (recommended)

OpenAI explicitly recommends XML for structuring multi-part inputs: *"using structured XML specs like `<[instruction]_spec>` improved instruction adherence on prompts and allows them to clearly reference previous categories and sections elsewhere in the prompt."*

This is one of the few places where Claude and GPT-5.x agree. If your prompt has multiple sections (instructions / context / examples / data), wrap each in a descriptive XML tag. Markdown headers also work but XML is more robust for cross-references.

### 7. Prompt caching: stable content first, dynamic tail last

OpenAI prompt-caching guidance: *"Move volatile content (e.g., user input, dynamic values) to the end. Even small changes in early tokens will invalidate exact prefix matching."*

This composes naturally with rule #6 from the long-context section in `claude.md` (long documents at the top, the question at the bottom). For cacheable agent loops, the system prompt + tool definitions go first (stable), the user message and any dynamic context go last.

### 8. Reasoning is a knob, not a wording trick

GPT-5.x exposes `reasoning_effort` with values `none` / `low` / `medium` / `high` / `xhigh` — and **`max` from GPT-5.6 onward**, positioned by OpenAI for the heaviest quality-first workloads. This is **the** lever for reasoning depth. "Think step by step" in the prompt body is essentially obsolete on GPT-5.x — it's a poor substitute for raising `reasoning_effort`, and on agentic frames where preambles or `phase: 'commentary'` updates already exist, it adds noise.

When the user complains "the model isn't reasoning carefully enough", check `reasoning_effort` first. The same advice applies to Claude (`effort` setting), but on GPT-5.x the effect is more dramatic.

### 9. Tone is direct by default — prompt for warmth if needed

GPT-5.5 default style: *"efficient, direct, and task-oriented… responses stay focused, behavior is easier to steer, and the model avoids unnecessary conversational padding."* If you need a warmer tone for end-user-facing copy, ask explicitly (e.g., "Match a supportive coach tone, acknowledge the user's framing before answering"). Verbosity is adjustable via `text.verbosity` (`low` / `medium` / `high`); use the parameter rather than prose where possible.

**Ask for tone as behavior, not as a label.** OpenAI's 5.6 guidance: *"Broad labels such as 'friendly' or 'empathetic' can be ambiguous. Describe the writing choices that define your product's tone."* So instead of "be friendly", write what the model should actually do — state the answer directly, name the trade-off before the recommendation, acknowledge a problem when one exists. Same class of fix as the methodological-anchor rule for personas (`../techniques.md` §6): name the behavior, not the adjective.

### 10. Preambles for tool-heavy / multi-step tasks

OpenAI recommends a short visible preamble before tool calls in long agentic workflows: *"a brief visible update that acknowledges the request and states the first step. Keep it to one or two sentences."* In the Responses API this is encoded with `phase: "commentary"` for intermediate updates and `phase: "final_answer"` for the completed answer.

When reviewing a Codex agent prompt that complains about lack of progress visibility, the wording fix is "tell the model to emit a one-sentence preamble before each tool call" — shape, not frequency.

### 11. CTCO is OpenAI's recommended structuring frame

Context → Task → Constraints → Output. From OpenAI's GPT-5.2 prompting guide. Each section is a separate `<...>` block. Useful when reviewing system prompts that have all four concerns mashed into one paragraph.

```xml
<context>
Persistent state, role, repository conventions, available tools.
</context>

<task>
The single atomic action the model should perform.
</task>

<constraints>
Negative scope, what NOT to do, deadlines, performance budgets.
</constraints>

<output>
Exact format expected — referenced shape if json_schema is used, or prose with explicit length/section spec otherwise.
</output>
```

If the prompt is short and self-contained, don't impose CTCO — it adds overhead. Use when the prompt is already mixing the four concerns and could benefit from separation.

### 12. Persona / "act as" is dying

Community consensus on GPT-5.x: outcome-first prompts beat persona-first prompts. "You are an expert at X, your task is Y" is weaker than just "Y" with the right success criteria. Strip "You are a senior X" lines from system prompts unless they encode a specific methodological lens (see `../techniques.md` §6 — methodological anchors are NOT this anti-pattern).

### 13. Visible reasoning is steerable but not a reliable audit trail

GPT-5.x can produce visible reasoning when asked, but — same caveat as for Claude — the displayed reasoning isn't a guaranteed reflection of the actual computation. Use it for UX (showing the user the model is working) but not as ground truth for debugging prompt behavior. Test on held-out cases.

### 14. State the autonomy boundary for each request

New in OpenAI's 5.6 guidance, and worth applying family-wide: *"Define what level of action each request authorizes so the model can continue safe, in-scope work without unnecessary pauses while stopping before external, destructive, costly, or scope-expanding actions."*

This is the wording fix for both failure directions in an agentic prompt — a model that stops to ask about every routine step, and a model that runs on into irreversible territory. One line covers both:

> Make routine, reversible changes inside the stated scope without checking in. Stop and ask before anything external, destructive, costly, or outside that scope.

---

## GPT-5.6 — Sol / Terra / Luna (frontier as of 2026-07-09)

OpenAI's current family: **`gpt-5.6-sol`** (flagship), **`gpt-5.6-terra`** (strong at lower price), **`gpt-5.6-luna`** (efficient, high-volume). The bare alias `gpt-5.6` routes to Sol. Model-card facts for Sol: 1,050,000-token context (input up to 922,000, output up to 128,000), knowledge cutoff 2026-02-16. The July 9 launch date comes from press coverage — OpenAI's card doesn't carry one.

Wording-wise the three variants are one family: same guidance, differing capability and price. Treat "which of Sol / Terra / Luna" as a cost decision, not a prompting decision — with the usual caveat that the cheapest tier is least forgiving of vague scope (same rule as `5.4-mini`, Haiku 4.5, Gemini Flash-Lite).

### Headline behaviors

- **Leaner system prompts are now a measured win, not a taste call.** See family rule #1 — 10–15% better eval scores at 41–66% fewer tokens. On a 5.6 review, deleting duplicated instructions outranks rewriting them.
- **More concise by default than GPT-5.5** — the guide states it plainly. This inverts a common carried-over instruction; see below.
- **Reasoning effort gains `max`** — the ladder is `none` / `low` / `medium` / `high` / `xhigh` / `max`.
- **Autonomy boundaries are the guide's recommended framing** for agentic work (family rule #14).
- **Tone is specified as behavior, not as an adjective** (family rule #9).

### Verbosity inverted — the finding most likely to already be in the prompt

GPT-5.6 *"tends to be more concise by default than GPT-5.5."* A "be brief" / "keep it short" / "no preamble" line carried over from a 5.5-era prompt now **over**corrects: it stacks on an already-terse default and produces answers too thin to be useful. The fix is not to rewrite the instruction but to move the default out of prose entirely — set `text.verbosity` (`low`/`medium`/`high`) and leave only task-specific length rules in the body ("the summary section is at most 5 bullets").

**This is a cross-vendor trap, not just a GPT one.** The same review, targeting Claude Opus 5, needs the opposite edit: Opus 5's defaults run long, it does not calibrate to task, and `effort` won't shorten it — concision has to be prompted (`claude.md § Claude Opus 5`). One `AGENTS.md` cannot carry a single verbosity setting for both. When the prompt is cross-vendor, state length as a task requirement ("at most 5 bullets") rather than as a global disposition ("be concise" / "be thorough") — a requirement lands the same on both defaults.

### Migration 5.5 → 5.6

OpenAI's rule for effort is explicit and unusual: *"Preserve your current reasoning effort as the baseline, then compare one level lower."* The expected direction of travel is **down**, not sideways — 5.6 does more with less thinking. (If you're on `none`, keep it as the latency baseline and also test `low` where the workflow benefits from reasoning or tool use.)

| Old 5.5-style | 5.6-style |
|---|---|
| The same instruction restated in three sections "so the model doesn't miss it" | State it once. Repetition is measured cost, not insurance. |
| "Be brief. Be concise. No preamble." | Delete; set `text.verbosity`. Keep only task-scoped limits ("at most 5 bullets"). |
| Long tool-usage prose in the system prompt | Simplify the tool descriptions themselves (family rule #3); the guide names tool-description bloat specifically. |
| `reasoning_effort` copied over from 5.5 unchanged | Test the same level **and one level lower**; keep the cheaper one if quality holds. |
| "Be friendly and helpful." | Describe the writing choices: state the answer first, name trade-offs, acknowledge a real problem instead of smoothing it over. |
| Silent about how far the model may go | Add the autonomy boundary line (family rule #14). |

### When 5.6-specific tuning matters most

- Any prompt stack ported forward from 5.5 or earlier — the trimming pass is where the measured gain sits.
- Agentic loops where the model either stalls asking permission or overreaches — the autonomy-boundary line is the targeted fix.
- Cost-sensitive high-volume work on Terra / Luna, where the token reduction compounds.

### When NOT to invest

- Cross-vendor `AGENTS.md` that must also serve Claude — trim what's redundant, but don't adopt 5.6's concision default as the file's global disposition.
- Prompts already lean and stable on 5.5: the fresh-baseline advice (family rule #2) is about *bloat*, not about rewriting healthy text.

---

## GPT-5.5 (previous frontier, 2026-04-23)

The prior flagship for agentic, long-context, and reasoning-heavy work — still widely deployed, and the version most in-the-wild prompt stacks are tuned for.

### Headline behaviors

- **Outcome-first interpretation**: rule #1 above hits hardest here. Verbose process prescriptions actively degrade output quality.
- **Lower hallucination rate**: ~60% reduction vs GPT-5.4 in OpenAI's stated benchmarks; a system-card claim, take with a grain of salt but the practical effect is visible.
- **Long-context strength**: ~272K context window with strong retrieval characteristics across the full window. Long-context techniques (documents at top, query at bottom, quote-grounding) still apply but are less load-bearing than on earlier 5.x.
- **Fresh-baseline migration**: the most aggressive of any GPT-5.x release. OpenAI explicitly recommends discarding 5.4-era prompt scaffolding rather than porting it.

### Wording fixes when migrating from 5.4 → 5.5

| Old 5.4-style | 5.5-style |
|---|---|
| "First, do X. Then do Y. Then check Z." | "The result must satisfy Z. X and Y are typical sub-steps; choose order yourself." |
| "Be thorough — examine each file." | "Coverage matters: report every issue, including low-confidence ones, tagged with confidence. A downstream filter will rank them." |
| Long block of "DO NOT do X / Y / Z" | One terse "Out of scope: X, Y, Z." line. Negative-only walls confuse 5.5 about what TO do. |
| "Think step by step." | (Delete.) Set `reasoning_effort=medium` or `high` instead. |
| Few-shot block of 5 examples for a reasoning-heavy task | Delete the examples; clarify the success criteria in 1–2 sentences. |
| "Return JSON like this: {…}" + sample | Move to `json_schema` with `strict: true`; system prompt drops the format prose. |

### When 5.5-specific guidance matters most

- Agentic loops with tool use (Codex CLI, Responses API agents). Outcome-first + tool descriptions doing the heavy lifting.
- Long-context analysis (>100K tokens). Stable-content-first ordering for caching; documents-at-top for attention.
- Migration scenarios from 5.4 or earlier — fresh baseline, not patches.

### When NOT to invest in 5.5-specific tuning

- Cross-vendor `AGENTS.md` — must work on Claude too; tune for the lowest common denominator.
- Tasks where 5.4 / 5.4-mini are good enough — don't optimize prompts for a model the user isn't actually using.

---

## GPT-5.4 (March 2026)

Strong reasoning + token efficiency. Has `gpt-5.4-mini` for lighter workloads.

### Behaviors that shape wording

- **`reasoning_effort` defaults to `none`**: explicit reasoning is opt-in. If your prompt expects multi-step reasoning, set `reasoning_effort` explicitly — don't try to force it via prompt.
- **`gpt-5.4-mini`**: same family character but less forgiving of vague prompts. Spell out scope and success criteria; don't expect mini to figure them out.
- **Outcome-first preference**: present, but less strict than 5.5. Step-by-step prescriptions are tolerated more on 5.4 than on 5.5.
- **Few-shot tolerance**: better than 5.5 — examples help more often, especially for output formatting where `json_schema` isn't an option.

### When 5.4-specific tuning helps

- Cost-sensitive production agents using `5.4-mini`. Specificity beats elegance.
- Tasks where 5.4 is "good enough" and migrating to 5.5 isn't worth a fresh-baseline rewrite.

### Migration 5.3 → 5.4

The new `phase` field appeared in the Responses API around this transition (`"commentary"` vs `"final_answer"` on assistant messages). If a multi-turn agentic prompt was written assuming flat assistant messages, dropping the phase distinction silently can degrade multi-turn behavior. Phase markers are an API/SDK concern but worth flagging in the review when the artifact is a Responses-API agent prompt.

---

## GPT-5.3 / GPT-5.3-Codex

`5.3-codex` is the long-term-support coding variant, optimized for agentic coding inside Codex CLI. Plain `5.3` is the general-purpose sibling.

### Behaviors that shape wording

- **Codex variant is more action-biased**: tuned for autonomous coding, less prone to "describe-instead-of-do" failure modes than general 5.3 on the same prompt. Action-mode prompting (`../techniques.md` §9) is rarely needed — but explicit scope ("touch only file X") is, because the model will run with it.
- **Less literal than 5.5**: closer in style to Claude Sonnet 4.6 — generalizes from one item to similar ones.
- **Tolerates older "act as expert" framing**: persona prompts haven't been deprecated as harshly as on 5.5. Still inferior to outcome-first, but not actively harmful.
- **Few-shot still useful**: 3–5 examples for output format steering is a normal default here, unlike on 5.5.

### When 5.3-codex specific tuning helps

- Long-running Codex CLI sessions where action bias is a feature. Add `simplify` / scope-limit snippets (`../techniques.md` §11) — overengineering risk is real.
- Prompts that lived under `5.3-codex` for months and are still being used; respect their accumulated tuning rather than porting to 5.5 wholesale unless the user is migrating.

---

## GPT-5.2 (December 2025)

Earlier in the family. Many in-the-wild prompts and templates are still tuned for it (especially CTCO-heavy structures from OpenAI's GPT-5.2 prompting cookbook).

### Behaviors that shape wording

- **CTCO structure was first formally taught here**: official cookbook recommends explicit Context / Task / Constraints / Output blocks. Still works on later 5.x but adds tokens; only keep when the prompt actually mixes the four concerns.
- **More tolerant of process-step prompts than 5.5**: a 5.2 prompt that says "first do X, then Y, then Z" runs cleanly. Same prompt on 5.5 is suboptimal.
- **Few-shot tolerance**: similar to 5.4 — examples help more often than they hurt.

### When the user's prompt is for 5.2

- Don't aggressively strip step-by-step structure — on 5.2 it's neutral-to-helpful.
- Don't push hard for `json_schema` migration unless the user's stack supports it; CTCO + prose output spec is a reasonable fallback.
- Suggest 5.4 / 5.5 migration as a separate, non-urgent follow-up if the prompt is healthy on 5.2.

---

## GPT-5.1 (legacy)

Earliest GPT-5 variant. Most current tooling has migrated past it; you'll mostly see 5.1 prompts in archived templates or older internal docs.

### Behaviors that shape wording

- **Less reasoning depth**: prompts that work on 5.1 often look heavily scaffolded with explicit step plans. That's a 5.1 adaptation, not a universal best practice.
- **Higher hallucination rate vs newer family**: anti-hallucination snippets (`../techniques.md` §12, §22) carry their weight here.
- **Shorter effective context**: long-document prompts that work on 5.5 may not fit cleanly on 5.1.

### When the user's prompt is for 5.1

- The prompt is probably overdue for a rewrite, but don't push if the user has a reason to stay on 5.1 (cost, latency, stability, internal policy).
- Treat 5.1 wording as a record of the model's quirks at that point — many of the patches the user added were answers to specific failures, even if they look stale by 2026 standards. Apply the **Prime directive** in SKILL.md.

---

## Universal / multi-version GPT-5.x prompts

A prompt that must work across two or more 5.x versions (e.g., a Codex agent prompt that runs on 5.3-codex *and* 5.5 depending on user config) is the GPT-side equivalent of a Claude universal prompt. Most production system prompts in 2026 fall here.

### Rules

**1. Write for the most literal version in scope.**

If 5.5 or 5.6 is in scope, default to outcome-first phrasing — earlier versions tolerate it; 5.5 and 5.6 punish the alternative. If **5.6** is in scope, also apply the leanness rule: a duplicated instruction that older versions merely ignored is measured cost here.

**1a. Verbosity defaults diverge inside the family.**

5.6 is terser out of the box than 5.5. A global "be concise" written for 5.5 overshoots on 5.6; a global "be thorough" written for 5.6 bloats 5.5. Express length as a task requirement ("at most 5 bullets", "one paragraph per finding") — that reads the same on every version — and leave the disposition to `text.verbosity`.

**2. Don't lean on 5.5-only features.**

- Don't assume 1M-token-class context behavior — 5.3 and earlier are tighter.
- Don't assume the lowest hallucination rate — keep anti-hallucination snippets (`../techniques.md` §12, §22).
- Don't strip few-shot examples that are load-bearing on 5.3 / 5.4 just because 5.5 prefers zero-shot. Keep them and trust 5.5 to ignore.

**3. Push output format into `json_schema` if the stack supports it.**

This is the most version-neutral lever. The system prompt becomes leaner regardless of which version reads it.

**4. Avoid hardcoding model names in the prompt.**

"You are GPT-5.5" locks you to the model. Prefer functional roles: "You are a code-review agent."

**5. Use `reasoning_effort` rather than wording for reasoning depth.**

Wording-based reasoning prompts ("think step by step") work inconsistently across versions. The API knob is consistent.

### Universal-GPT checklist

Before calling a prompt "universal across GPT-5.x", verify:

- [ ] No hard dependency on 5.5-only context length (>200K) or 5.5-only hallucination rate
- [ ] Output contract is in `json_schema` if stack supports it; otherwise format is described once, not enforced through few-shot
- [ ] Reasoning depth is configured via `reasoning_effort`, not "think step by step" in prose
- [ ] No 5.4-era process-step prescription that 5.5 would treat as noise (or it's tagged "step suggestions, not requirements")
- [ ] No model name hardcoded in the system prompt
- [ ] Tool-specific guidance is inside tool descriptions, not the system prompt
- [ ] Stable content first; volatile / user-specific content at the end

---

## Cross-vendor universal prompts (Claude + GPT-5.x)

The hardest case: an `AGENTS.md` or system prompt meant to work on **both** Claude Code (any current Claude) and Codex CLI (any current GPT-5.x). The two vendors have **opposite defaults** on several axes; cross-vendor wording must be acceptable to both, which usually means picking the stricter constraint.

### The opposite-default table

| Axis | Claude default | GPT-5.x default | Cross-vendor wording |
|---|---|---|---|
| Process vs outcome | Tolerates / appreciates step naming, especially Opus 4.7 | Treats step prescription as noise, especially 5.5 | Outcome-first wording with steps as "typical sub-tasks, choose order yourself" |
| Few-shot examples | 3–5 diverse examples = standard | Often hurts reasoning models — zero-shot default | Skip few-shot unless format is genuinely strict; if needed, keep to 1–2 |
| Tool guidance location | System prompt is fine | Strongly prefer inside tool description | Move tool-specific guidance into tool descriptions; keep system prompt cross-tool only |
| Reasoning prompts | "Verify against criteria" works; "think harder" doesn't | `reasoning_effort` parameter; wording prompts mostly inert | Drop "think step by step" entirely; ask for specific verification or rely on the parameter |
| Persona / role | Functional one-line role useful | Outcome-first usually beats persona | Use functional role only when it encodes a methodological lens; otherwise drop |
| Subagent spawning | Opus 4.7 spawns fewer; may need explicit "do spawn" | Codex spawns subagents you defined; rarely speculative | Be explicit when you want delegation; don't assume default behavior |
| Output format | Prose constraints work | Push to `json_schema` when possible | Prefer `json_schema` for both; falls back to prose constraints if API doesn't support it |
| Aggressive emphasis | Overuse causes overtriggering on 4.5+ | Mostly inert / mild noise | Reserve ALL-CAPS / "CRITICAL:" for genuine safety invariants only |
| **Response length default** | **Opus 5 runs long and does not calibrate** — concision must be prompted, `effort` won't shorten it | **5.6 is already terse** — a carried-over "be brief" overcorrects | State length as a **task requirement** ("at most 5 bullets"), never as a global disposition ("be concise" / "be thorough") |
| Repeated instructions | Tolerated; repetition near the end is even recommended on Opus 5 for concision | **Measured cost on 5.6** (leaner prompts: +10–15% score, −41–66% tokens) | State each rule once, in the section where it belongs; if a rule must be echoed, keep the echo to one short line |

### Rules for cross-vendor `AGENTS.md`

**1. Outcome-first wording, not step prescription.**

The worst case is the cross-vendor case for steps: if you write step-by-step, 5.5 will read it as noise; if you write pure outcome, Opus 4.7 may not generalize the way you expect. The compromise: state the outcome first, then mention sub-steps as *typical* not *required*.

> Fix the failing CI pipeline. Typical sub-tasks: read the most recent failed log, identify the failing job, propose a fix, run tests locally before pushing. Adapt as needed.

This phrasing works on both sides because the outcome ("fix the failing CI pipeline") leads, and the steps are framed as guidance rather than constraint.

**2. Drop persona / role lines unless they're methodological anchors.**

"You are a senior engineer" adds attention noise on GPT-5.5 and isn't load-bearing on Claude. Drop it. "Apply systems-thinking lenses: 2nd/3rd-order effects, 10-year horizon" is a methodological anchor — keep it on both sides.

**3. Watch the Codex 32 KiB cap.**

If the `AGENTS.md` is shared with Codex (i.e., it lives at the project root with no `.codex/` override), it's subject to Codex's `project_doc_max_bytes` (32 KiB by default). Claude Code has no equivalent hard cap but suffers context rot past ~300 lines. The strictest constraint wins: keep it short.

**4. Use markdown headers, not heavy XML, for structure.**

Both vendors handle markdown headers and XML well. Markdown is more readable in version control and travels better between tools that parse it (`agents.md` adopters, some IDE integrations). Reserve XML for inline structuring within sections, not as the primary skeleton.

**5. Tool descriptions over system prompt for tool guidance.**

This is OpenAI's strong recommendation; Anthropic tolerates either. Pick OpenAI's stricter rule for cross-vendor — it costs nothing on Claude and matters on GPT-5.x.

**6. Avoid model-name pinning.**

"When using Opus 4.7, do X" or "GPT-5.5 should Y" inside an `AGENTS.md` defeats the purpose of cross-vendor. Functional descriptions only.

### Cross-vendor universal checklist

Before calling an `AGENTS.md` cross-vendor:

- [ ] Outcome-first phrasing dominates; steps are guidance, not requirement
- [ ] No model-name pinning
- [ ] Persona lines either dropped or carry methodological anchors
- [ ] Tool-specific guidance migrated into tool descriptions
- [ ] Anti-hallucination snippets (`../techniques.md` §22) explicit (not assumed from a low-hallucination model)
- [ ] Total file size well under 32 KiB (target <8 KiB for the file's load-bearing rules)
- [ ] Aggressive emphasis used on safety invariants only
- [ ] No "think step by step" in the body
- [ ] Output contracts pushed to `json_schema` where the stack supports it
- [ ] Universal-Claude checklist (in `claude.md`) and universal-GPT checklist (above) both pass

When the two checklists conflict, the **cross-vendor wording table** (above) decides.

---

## Cross-model wording principles (GPT side)

Mirror of the section in `claude.md`. These hold across all GPT-5.x versions:

### Use `reasoning_effort` over wording for reasoning depth

`reasoning_effort` (`none` / `low` / `medium` / `high` / `xhigh`, plus `max` from 5.6) is the load-bearing knob. Wording-based reasoning prompts are inferior across the family. When migrating a version forward, test **one level lower** than the current setting — that's OpenAI's own 5.6 migration rule, and the direction newer versions keep moving.

### Prefer "verify" / "evaluate" / "consider" over "think"

Same as on Claude: "think harder" doesn't work; "verify your answer against these criteria before finishing" does.

### Aggressive emphasis is mostly inert noise

GPT-5.x doesn't overtrigger on "CRITICAL:" the way Claude 4.5+ does, but it doesn't reliably elevate the rule's importance either. Use sparingly; rely on structure (XML, headers) and outcome wording instead.

### Prompts age fast

OpenAI ships frequent minor versions and the prompting style shifts more between them than Claude generations do. A prompt that's 6 months old should get a fresh-eyes review on every model upgrade — don't assume the old wording still pulls its weight.
