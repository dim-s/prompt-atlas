# Model-specific wording differences — OpenAI GPT-6.x and GPT-5.6

What actually changes about how you should PHRASE prompts for each current OpenAI model. Companion to `claude.md` (Claude). This reference is about wording — not API parameter values, context sizes, or pricing; a parameter or context fact appears only where it changes how a prompt is designed.

**Lineup covered here (read 2026-10-31).** Current: GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol, GPT-6 Luna (§ GPT-6 family); the voice model GPT-Live 1 (§ GPT-Live 1). Previous generation: GPT-5.6 Sol / Terra / Luna (§ GPT-5.6). GPT-5.5 and older live in `../archive/models/gpt.md` (not maintained). There is **no GPT-6.1 Astra and no GPT-6.1 Luna** — their model pages return 404.

**Retirements (one note; the dates matter only because the target model changes).** API: `gpt-5`, `gpt-5-mini`, `gpt-5-nano`, `gpt-5-pro`, `o3`, `o3-pro` shut down 2026-12-11 (→ `gpt-5.6-*`); GPT-4o / GPT-4.1 2027-01-14 (→ `gpt-6-luna`); `gpt-5.1`, `gpt-5.3-codex`, `gpt-5.4-nano` 2027-04-01 (→ `gpt-6-sol` / `gpt-6-luna`). GPT-5.6 and the GPT-6 family are not on the deprecations page. Codex with ChatGPT sign-in (API-key sessions unaffected): GPT-5.5 retired 2026-10-14 (→ `gpt-6.1-sol`; Free / Go → `gpt-6-luna`), `gpt-5.4` / `gpt-5.4-mini` 2026-08-31, `gpt-5.2` and `gpt-5.3-codex` already deprecated. A Codex prompt or subagent still pinned to one of these runs on a different model than it was tuned for.

When the artifact runs in OpenAI Codex CLI specifically (AGENTS.md, Codex subagent, Codex skill), also read `../agentic-systems/codex.md` — Codex layers its own behavior on top of the underlying model and that interacts with prompt design.

---

## Family-wide rules (GPT-6.x and GPT-5.6)

These come from OpenAI's 5.x guides (5.2 → 5.6) and hold for 5.6. The GPT-6 guide neither restates nor contradicts most of them (outcome-first, tool guidance in descriptions, output contracts in schemas), so they carry over to GPT-6.x **by inference**. Where GPT-6 changes a default — length and format, clarifying questions, delegation — its section says so. Version-specific notes follow below.

### 1. Outcome-first beats process-prescription

OpenAI's prompt-guidance is explicit: GPT models perform best when you describe the **target outcome and success criteria**, not the steps to get there. From the GPT-5.5 guide: *"GPT-5.5 treats detailed step-by-step instructions as interference: redundant instructions create noise, narrow the solution space, and make responses overly mechanical."*

**On GPT-5.6 OpenAI put a number on it.** The latest-model guide reports that *"configurations with leaner system prompts improved evaluation scores by roughly 10–15% while reducing total tokens by 41–66% and cost by 33–67%."* That makes trimming the single highest-value edit on a 5.6-targeted review — not a stylistic preference. The guide's operational rule: remove repeated instructions, simplify tool descriptions, and state each instruction **once**.

| Weak (process-heavy) | Strong (outcome-first) |
|---|---|
| "First read the file, then identify the bug, then propose a fix, then write a test, then implement..." | "Fix the bug in `src/auth/session.py` that causes logout to fail after token refresh. Verification: the existing test suite passes plus a new regression test you add." |
| "Step 1: validate input. Step 2: query database. Step 3: format response." | "Endpoint must return 200 with the matching record on valid input, 404 on missing record, 400 on malformed input. Choose the implementation order yourself." |

This is the single biggest divergence from Claude defaults: Claude often appreciates explicit step naming; GPT reads it as noise.

### 2. Treat each new GPT version as a fresh baseline

OpenAI's GPT-5.5 migration guide is unusually direct: *"treat it as a new model family to tune for, not a drop-in replacement… begin migration with a fresh baseline instead of carrying over every instruction from an older prompt stack."*

Practical impact: a system prompt tuned across several older versions — with patches accumulated against their overengineering or hallucination quirks — often **underperforms** on the newer model versus a leaner zero-baseline rewrite. The accumulated patches each cost attention and now address problems the new model doesn't have.

When you see a long system prompt being ported across versions, flag this as a candidate for a from-scratch rewrite, not incremental tuning.

### 3. Tool-specific guidance lives in tool descriptions, not the system prompt

OpenAI's official guidance: *"Put most tool-specific guidance in the tool descriptions themselves: what the tool does, when to use it, required inputs, side effects, retry safety, and common error modes. Add tool-specific context to system instructions only when it applies across tools."*

This is a stronger split than Anthropic recommends. On GPT, system prompts crowded with per-tool usage hints (when to call X, what to pass to Y) are a sign the wording belongs inside the tool's `description` field instead.

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

This is one of the few places where Claude and GPT agree. If your prompt has multiple sections (instructions / context / examples / data), wrap each in a descriptive XML tag. Markdown headers also work but XML is more robust for cross-references.

### 7. Prompt caching: stable content first, dynamic tail last

OpenAI prompt-caching guidance: *"Move volatile content (e.g., user input, dynamic values) to the end. Even small changes in early tokens will invalidate exact prefix matching."*

This composes naturally with rule #6 from the long-context section in `claude.md` (long documents at the top, the question at the bottom). For cacheable agent loops, the system prompt + tool definitions go first (stable), the user message and any dynamic context go last.

### 8. Reasoning is a knob, not a wording trick

Every current GPT text model exposes `reasoning_effort` (`reasoning.effort` in Responses; the GPT-Live 1 model page names no reasoning control), and the ladder is per model: `none` / `low` / `medium` / `high` / `xhigh` / `max` on GPT-5.6, GPT-6 Sol, GPT-6 Luna and GPT-6.1 Terra; `low` … `max` with **no `none`** on GPT-6 Astra and GPT-6.1 Sol (table in § GPT-6 family). This is **the** lever for reasoning depth. "Think step by step" in the prompt body is essentially obsolete — it's a poor substitute for raising `reasoning_effort`, and on agentic frames where preambles or `phase: 'commentary'` updates already exist, it adds noise.

When the user complains "the model isn't reasoning carefully enough", check `reasoning_effort` first. The same advice applies to Claude (`effort` setting), but on GPT the effect is more dramatic.

### 9. Tone is direct by default — prompt for warmth if needed

GPT-5.5 default style: *"efficient, direct, and task-oriented… responses stay focused, behavior is easier to steer, and the model avoids unnecessary conversational padding."* GPT-5.6 is terser still; **GPT-6 Astra swings back** to detailed, formatted answers (§ GPT-6 Astra), and the other GPT-6 models are undocumented on this axis. If you need a warmer tone for end-user-facing copy, ask explicitly (e.g., "Match a supportive coach tone, acknowledge the user's framing before answering"). On 5.6, verbosity is adjustable via `text.verbosity` (`low` / `medium` / `high`); use the parameter rather than prose where possible.

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

Community consensus on GPT (formed on 5.x): outcome-first prompts beat persona-first prompts. "You are an expert at X, your task is Y" is weaker than just "Y" with the right success criteria. Strip "You are a senior X" lines from system prompts unless they encode a specific methodological lens (see `../techniques.md` §6 — methodological anchors are NOT this anti-pattern).

### 13. Visible reasoning is steerable but not a reliable audit trail

GPT can produce visible reasoning when asked, but — same caveat as for Claude — the displayed reasoning isn't a guaranteed reflection of the actual computation. Use it for UX (showing the user the model is working) but not as ground truth for debugging prompt behavior. Test on held-out cases.

### 14. State the autonomy boundary for each request

New in OpenAI's 5.6 guidance, and worth applying family-wide: *"Define what level of action each request authorizes so the model can continue safe, in-scope work without unnecessary pauses while stopping before external, destructive, costly, or scope-expanding actions."*

This is the wording fix for both failure directions in an agentic prompt — a model that stops to ask about every routine step, and a model that runs on into irreversible territory. One line covers both:

> Make routine, reversible changes inside the stated scope without checking in. Stop and ask before anything external, destructive, costly, or outside that scope.

---

## GPT-6 family — lineup, and how far the guide's prompts reach

Five models are live in the API: `gpt-6-astra` (top), `gpt-6.1-sol` (*"Near-Astra performance for complex work at a lower cost"*), `gpt-6.1-terra` (*"a fast, efficient model for visual reasoning, precise computer use, and clear, natural writing"*), `gpt-6-luna` (focused, high-volume work), and `gpt-6-sol`, which stays callable (*"See GPT-6.1 Sol for the newer Sol model"*). Name collision: GPT-5.6 also had a Sol, a Terra and a Luna — a review that says "Sol" or "Terra" must name the generation.

| Model | `reasoning_effort` | `temperature` / `top_p` / `top_logprobs` | Tool calling via Chat Completions |
|---|---|---|---|
| GPT-6 Astra | `low` … `max`, **no `none`** (model page names no default) | never — the guide drops them whenever effort isn't `none` | no — Responses API only |
| GPT-6.1 Sol | `low` … `max`, default `medium`; **no `none`, no `minimal`** | never (same rule, inference) | no — Responses API only |
| GPT-6.1 Terra | `none` … `max`, default `medium` | only at `none` (inference from the same rule) | no — Responses API only, even at `none` |
| GPT-6 Sol, GPT-6 Luna | `none` … `max`, default `medium` | only at `none` | only with `reasoning_effort: "none"` |

**The guide's prompt block is family-wide, its observations are Astra's.** Since 2026-10-06 the latest-model guide ("Using GPT-6") opens its prompting section with: *"Use the following prompts as a starting point across the GPT-6 model family. They address behavior observed with GPT-6 Astra; evaluate them with your chosen model and workload."* There are no model-specific wording sections for Sol, Terra or Luna. So the behavior claims in § GPT-6 Astra are facts about Astra only; the snippets there are the vendor's starting point for every GPT-6 model — apply them on Sol / Terra / Luna when testing shows the symptom they fix, and don't strip 5.6-era content from a Sol / Terra / Luna prompt on Astra's authority.

Consequences for a review: (1) a claim like "GPT-6 has no `none`" or "GPT-6.1 has no `none`" is wrong per model — Terra, 6 Sol and 6 Luna have it; (2) "answer without reasoning" is not a prompt job — on Terra / 6 Sol / 6 Luna the off-switch is `reasoning_effort: "none"`, on Astra and 6.1 Sol there is none and the floor is `low`; (3) a prompt stack that sets `temperature` together with effort above `none` is a finding on every GPT-6 model; (4) an artifact that assumes Chat Completions tools works only on 6 Sol / 6 Luna at `none`.

**Shared features (whole family, harness concerns — surface, don't write wording for them):** async tool calling and mid-turn steering (Responses API), `configuration_update` to change reasoning effort mid-conversation without rewriting the cached prompt prefix, computer use, structured outputs, multi-agent orchestration, prompt caching with `prompt_cache_options.ttl` (e.g. `"30m"`) in place of `prompt_cache_retention`.

Sources: developers.openai.com/api/docs/guides/latest-model.md ("Using GPT-6"), model pages `models/gpt-6-astra.md`, `gpt-6.1-sol.md`, `gpt-6.1-terra.md`, `gpt-6-sol.md`, `gpt-6-luna.md`, `guides/reasoning.md` — all read 2026-10-31 (raw). `models/gpt-6.1-astra.md` and `models/gpt-6.1-luna.md` → 404 (read 2026-10-31). The 2026-10-06 date of the family-wide framing comes from Wayback raw snapshots of the guide (2026-09-15 vs 2026-10-06).

---

## GPT-6.1 Sol (2026-09-29) and GPT-6.1 Terra (2026-10-26)

Neither model has wording guidance of its own; the family starting point above applies. What changes for a prompt is out-of-band but wording-visible.

### GPT-6.1 Sol — delta on GPT-6 Sol

- **`none` and `minimal` are gone.** *"GPT-6 Astra and GPT-6.1 Sol do not support `none`; use `low` instead."* and *"If your existing request uses `minimal`, start with `low` and compare results on representative tasks."* A 6 Sol stack that ran on `none` for latency has no equivalent here — that is a model choice (6.1 Terra / 6 Luna), not a prompt rewrite.
- **No tools through Chat Completions at any effort** (6 Sol allowed them at `none`): *"Chat Completions does not support function calling with GPT-6 Astra or GPT-6.1 Sol."*
- **Context in Codex is half the API window:** *"GPT-6.1 Sol uses a default context window of 500K tokens in Codex and ChatGPT Work."* A prompt architecture that loads close to the 1M API window (whole-repo dumps, very long instruction stacks) does not fit the Codex default. 6.1 Sol is also the Codex replacement for the retired GPT-5.5 on paid plans.
- System-card FYI, not a wording rule: told *"Do not reason about this question in analysis; think about anything else"*, *"GPT‑6.1 Sol generates no analysis-channel text, but makes question-related tool calls in commentary"* — an instruction not to think does not stop the work from happening in tool calls.

### GPT-6.1 Terra — the only 6.1 model with `none`

- Effort `none` … `max`, default `medium`: *"Start with `medium` and increase it if the improvement in quality is worth the added time and tokens."*
- **Migration from GPT-5.6 Terra keeps the effort level** — *"compare both models on representative images and workflows using the same reasoning effort"* — unlike the 5.5 → 5.6 rule of testing one level lower. *"If your application calls tools through Chat Completions, migrate those requests to the Responses API."*
- Role: images, screens, computer use, drafting and revising text. Codex's own example prompt for it shows the pattern for visual tasks — name what to cite and ask for uncertainty: *"Review these dashboard screenshots and summarize the main changes. Refer to the chart labels and values, and flag anything you can't read confidently."*
- System card: *"GPT-6.1 Terra makes substantially fewer factual errors and is significantly less likely to reproduce user-reported hallucinations than GPT-6 Sol, achieving performance on par with GPT-6.1 Sol."* Not a reason to drop anti-hallucination lines from a prompt that also runs on other models.
- Codex: *"GPT-6.1 Terra requires Codex CLI 0.159.0 or later."*

Sources: model pages `models/gpt-6.1-sol.md`, `models/gpt-6.1-terra.md`, `guides/latest-model.md`, `guides/reasoning.md`, `api/docs/changelog.md` (release dates) — developers.openai.com, read 2026-10-31 (raw); system-card addenda deploymentsafety.openai.com/gpt-6-1-sol (published 2026-09-29) and /gpt-6-1-terra (published 2026-10-26), read 2026-10-31 (raw HTML); learn.chatgpt.com/docs/models.md (Codex notes), read 2026-10-31 (raw). Not read: openai.com/index/introducing-gpt-6-1-terra (served a 500 error body).

---

## GPT-6 Astra (September 2026 — top of the GPT-6 generation)

`gpt-6-astra` — OpenAI's first GPT-6 model and still the top one; the system card is dated September 3, 2026. The family-wide rules above carry over *by inference* (see the family-rules note). What the guide **does** document is five behavior patterns *"observed with GPT-6 Astra"*, and three of them run opposite to what a 5.6-tuned prompt assumes. The snippets below are offered for the whole GPT-6 family (§ GPT-6 family); the behavior claims are Astra's.

The guide's own summary of the new default: *"When instructions leave room for interpretation, it uses the context it has to fill in routine gaps and asks focused questions when the answer could change the outcome."*

### What changed from GPT-5.6 — the review table

| Axis | GPT-5.6 | GPT-6 Astra | Wording move |
|---|---|---|---|
| Clarifying questions / stopping | autonomy-boundary line (family rule #14) covers both failure directions | more likely to ask and to stop *"when the user may expect it to make reasonable assumptions and persist"* | add the initiative snippets below; name what is already authorized |
| Default length and format | terser than 5.5 | *"tends toward detailed, formatted responses and may use recurring phrases across sessions"* | **inverted** — specify prose and structure as described behavior |
| Subagent delegation | spawns what's defined | *"may delegate less often than desired"* | encourage, and say when — opposite of Claude Fable 5.1 / Opus 5, which need boundaries |
| Instruction files | — | *"more sensitive to instructions contained in skills and other files, such as `AGENTS.md`"* | audit loaded skills; state user-over-skill precedence |
| Testing on small changes | — | thorough by default — broader tests than small tasks need | calibrate testing |
| Sampling | tunable | `temperature`, `top_p`, `top_logprobs` unsupported | remove from code; steer tone and variety with words |
| Reasoning effort | `none` … `max` | `low` … `max` — `none` unsupported | from `none` / `minimal`, start at `low` (API) |

### Initiative and follow-through — the collaborator default

The guide calls the model *"designed to be a more effective collaborator"* and is direct about the side effect on autonomous runs. Its starting prompt for autonomous work:

> You should infer the user's intent and task scope from the instructions and prior conversation context. Your job is to bias towards action and carry the user's intended task to completion.
>
> When the user expresses intent to perform new work or fix an existing issue, persist until the user's intended goal is complete. Progress autonomously towards the user's goal (e.g. creating isolated worktrees / checkouts if needed, resolving merge conflicts, read-only actions, creating draft PRs etc.) unless they are clearly destructive or irreversible.

When the request implies authorization but reads as a question:

> When the user's prompt indicates a request for action, such as "can you...", "I want to...", "help me..." and similar expressions, treat these as instructions to do the work and take action. Do not stop at acknowledging capability (e.g. "Yes…"), proposing a plan, or offering to continue. Do not settle for a partial or "helpful enough" solution that does not fully satisfy the user's task to save time, effort or tokens. If a task requires sustained work, complete all the necessary work until the intended outcome is fulfilled.

And to move approval to the end of the work instead of the start:

> Before asking the user clarifying questions, you should complete the work that is already authorized from context and necessary to make the proposed action concrete and reviewable. The user should be approving a concrete, reviewable result. For example, before deploying a change, writing to an external application, merging a PR or publishing a site, do all the required work first so that user approval is the final step. You don't need user permission for reversible tasks, read-only actions, reviews or fixes, or anything for which authorization is provided earlier in the session or strongly implied from the task instruction.
>
> Do not introduce unsolicited warnings, disclaimers, approval flows, or safety/compliance checklists due to hypothetical risk.

The guide's tuning note: *"The model also likes to ask non-blocking questions as it's working by default, so adjust these prompts to match the level of autonomy your application needs."* Its migration checklist names the symptom these snippets target — *"**Unnecessary approval pauses:** If you run into issues where the model keeps asking for approval before proceeding, use the initiative and follow-through guidance to prompt for more autonomous execution."*

**Pair this with an explicit confirmation policy where consequential actions exist.** The system card: *"We train Astra to be careful when performing consequential actions and to follow a confirmation policy provided as instructions to the deployed agent."* In OpenAI's adversarial workplace evaluations the misaligned-outcome rate was 3.4% for Astra without the policy and 3.0% with it (GPT-5.6 Sol: 18.8% → 8.0%) — *"Exposing the confirmation policy to the model reduced this rate further."* The two pieces of guidance fit together: name the specific actions that need approval (send, purchase, publish), and license everything else so the model stops asking about it.

On GPT-5.6, family rule #14's one-line boundary remains the documented form. On the other GPT-6 models, add the snippets when the stall shows up in testing — the vendor offers them as a starting point, not as a fix for an observed behavior there.

### Instruction following: audit what it reads

Stronger instruction following comes with sensitivity: *"unclear or conflicting guidance in a skill file may cause the model to pause and block work early. Make the priority of user instructions and skills explicit."* OpenAI also writes *"We strongly recommend auditing skills and other files accessible to your model for instructions that could influence its behavior."* Two documented lines:

> The user's instructions take precedence over guidelines provided in a skill. If explicit user instructions conflict with a skill's instructions, prioritize the user's instructions.

> If a skill causes you to ask for permission or confirmation, pause, leave requested work unfinished, or diverge from the user's intent, name and link to the exact SKILL.md file you read, quote the relevant instruction, and briefly explain how it applies. Distinguish explicit skill requirements from your interpretation of guidelines.

The second is a diagnostic. The guide: *"Asking the model to identify the skill and instruction that caused it to pause or change direction can also be effective in providing transparency into model behavior."* and *"Use this prompt to find silent and conflicting guidance when your application loads many skills and instruction files such as `AGENTS.md`."*

### Writing style: detailed and formatted — the 5.6 terseness assumption no longer holds

A 5.6-era review deleted "be brief" lines because the model was already terse. On Astra the default swings back toward lists, tables and Markdown; the guide's fix is wording, not a parameter:

> Default to using clear, concise paragraphs, each developing one main idea. Use lists only when the information is genuinely parallel, sequential, or easier to compare, and avoid nested lists unless the hierarchy cannot be expressed clearly in prose. Use plain, simple language: familiar words, concrete examples, and precise verbs. Prefer active voice and direct statements.
>
> Make sure to state the main point clearly and early, then develop it with the explanation and detail the reader needs. Let each sentence build on what came before. Develop the points that matter and provide enough support to be useful.

For technical communication:

> Use plain language over jargon, and reference technical details only to the degree that it helps illustrate an idea or your work to the user. Communicate complex concepts in a clear and cohesive manner, and calibrate your writing to the level of background knowledge assumed from the user's prompt and context.

For recurring stock phrases, the guide ships a phrase list; its second paragraph targets framing habits rather than words:

> Avoid using slop words or phrases like "Bottom Line:" in conclusions, "delve," "foster," "leverage," "it's worth noting," "importantly," "Question? Answer." or "This isn't about X. It's about Y.", "genuinely" or hyphenated compound descriptions and adjectives. Do not use concluding summary statements such as "In short:..", "The simplest mental model is:...".
>
> State the intended action directly. Avoid adding what you won't do, what will remain unchanged, or how you'll separate or categorize results. Do not use contrastive framing such as "X, not Y" that introduces an unprompted alternative that the user didn't ask about. Avoid invented compound labels like "exact-head checks" and "editorial-row layouts", vague qualifiers, and canned transitions; use plain verbs and prepositions to state the actual relationship directly.

**Cross-vendor consequence.** Claude Fable 5.1 formats *less* by default, Astra *more*. A shared `AGENTS.md` can't carry one formatting disposition — state structure per deliverable ("findings as a numbered list, rationale in prose"), the same move as stating length as a task requirement.

### Subagents: encouragement, not boundaries

> If at any point you can parallelize work by delegating tasks to another agent (no matter if you are the root or subagent), you should do so using collaboration tools if it could save time or improve quality.

> Messages that you send to other agents and your final answer may be read by a human, so ensure they are legible. Always put proper spaces between words and/or numbers.

The second line exists because *"Messages between agents may contain grammar or spacing errors."* The guide's tuning note: *"The model tends to respond well to prompting for how and when it should delegate work to subagents, so tune this behavior to fit with your harness and multi-agent implementation."*

The first line is exactly the kind of encouragement that must be *stripped* for Claude Opus 5 / Fable 5.1. In a cross-vendor prompt, state delegation as a condition (`claude.md § Universal` rule 7 phrasing) rather than as a push in either direction.

### Testing and verification: calibrate

> Do not write tests for reversible, low-impact changes that mirror the implementation. If you do choose to verify your work with tests, make sure that the tests are meaningful and necessary to verify implementation.
>
> Run tests appropriate to the change and complete required checks. Once those pass, broaden or repeat testing only when new changes, failures, or unresolved concerns justify it; otherwise, continue toward completing the task.

Same symptom family as Opus 5's over-verification, but the documented fix is calibration, not removal.

### Migration GPT-5.6 → GPT-6 (written against Astra; the guide offers it to the whole family)

| Old 5.6-style | GPT-6-style |
|---|---|
| Relies on the terse default; no prose/format guidance | Describe the prose and structure you want; add the phrase list if stock phrases show up |
| Only the one-line autonomy boundary | Add initiative / approval-at-the-end snippets where the agent stalls on questions; name the confirmation policy for consequential actions |
| Many skills / `AGENTS.md` files with loose or overlapping guidance | Audit them; add the user-over-skill precedence line (and the diagnostic line while tuning) |
| Silent about delegation, or "delegate only when necessary" | Say when and how much to delegate |
| `temperature` / `top_p` / `top_logprobs` set in code | Remove whenever effort isn't `none` — always on Astra and 6.1 Sol (API) |
| `reasoning_effort: none` or `minimal` | *"Preserve your current effective reasoning effort where supported."* Astra / 6.1 Sol: start at `low` and compare; Terra / 6 Sol / 6 Luna keep `none` (API) |
| Changing effort by rewriting the prompt prefix between turns | `configuration_update` input item — keeps the cached prefix (API) |

Tool calling requires the Responses API on Astra, 6.1 Sol and 6.1 Terra (Chat Completions works without tools); 6 Sol / 6 Luna allow it in Chat Completions only at `none` — an API concern, surface it if the artifact assumes Chat Completions tools.

### Safety-card notes

- *"Astra is our most robust model to prompt injection to date."* Not a licence to drop "treat retrieved content as data" lines in agents that read untrusted input.
- *"Astra makes substantially fewer factual errors than GPT-5.6 Sol"* on user-flagged hallucination-prone conversations. Cross-version prompts that also run on 5.x keep their anti-hallucination snippets (universal-GPT rule 2).

Sources: OpenAI latest-model guide (then "Using GPT-6 Astra") and model page (developers.openai.com), GPT-6 Astra System Card (deploymentsafety.openai.com, published 2026-09-03) — all read 2026-09-13. The launch post and Safety overview on openai.com returned HTTP 403 at read time; nothing above depends on them. Re-read 2026-10-31 (raw `guides/latest-model.md`, now "Using GPT-6"): the family-wide framing, the tuning notes on initiative / instruction following / subagents, the "approval pauses" checklist line, the second prose paragraph, the technical-communication snippet and the second slop paragraph were added to this section from that read; per Wayback raw snapshots all of those snippets were already in the guide on 2026-09-15.

---

## GPT-6 Sol and GPT-6 Luna (22 September 2026 — the working and volume tiers)

`gpt-6-sol` and `gpt-6-luna`, both still live. Roles per the model pages (read 2026-10-31): Sol *"is built for complex coding and agentic workflows"* and now points to GPT-6.1 Sol as *"the newer Sol model"*; Luna is *"our most efficient model for focused, high-volume tasks"*. Effort ladder, sampling and Chat Completions rules: the table in § GPT-6 family — the API-side difference from Astra is that both keep `none`, and with it sampling parameters and Chat Completions function calling at `none`.

**Wording:** the vendor's guidance for these two is the family-wide starting point (§ GPT-6 family); nothing first-party says Sol / Luna share Astra's collaborator default, detailed-and-formatted style, sensitivity to `AGENTS.md`, or under-delegation. Test the § GPT-6 Astra snippets against the symptom; until OpenAI publishes model-specific guidance or you have measured one, don't tune further.

Sources: OpenAI latest-model guide (developers.openai.com/api/docs/guides/latest-model, read 2026-09-24, two passes through WebFetch that agree); TechCrunch 2026-09-22 (launch report); model pages `models/gpt-6-sol.md`, `models/gpt-6-luna.md`, read 2026-10-31 (raw). Not read: openai.com/index/introducing-gpt-6-sol-and-luna (403).

---

## GPT-Live 1 — full-duplex voice prompts (`gpt-live-1`, GA 2026-09-10)

A voice model that *"can listen and speak at the same time, and delegate reasoning and tool use to a backend agent."* The prompt splits in two: *"Use `session.instructions` for the assistant's role, speaking style, and when to ask the backend for help. Give the backend model or agent the procedures and tools for tasks such as looking up an order or changing a booking."* What differs from a text or Realtime voice prompt:

- **A model-specific control phrase.** *"For `gpt-live-1`, use the instruction `Generate <|SpawnThinking|>` when the model should request backend work, rather than "delegate to the backend.""* And *"A spoken promise such as "I'm checking that" does not confirm that backend work has started."* The migration guide's template still shows `Delegate to the backend when` labels; for `gpt-live-1` the prompting guide's `<|SpawnThinking|>` labels are the documented form.
- **Fixed headings.** *"Keep the template's `Backchannel policy`, `Interruption policy`, and `Delegation policy` headings, and customize the text beneath them."* Under `Delegation policy`: *"Keep the template's three labels: `Backend tools`, `Generate <|SpawnThinking|> when`, and `Do not generate <|SpawnThinking|> when`."* Rename them in a review only with a test behind it.
- **Less scripted wording.** *"Describe the conversational behavior you want, and let GPT-Live choose the wording for ordinary replies."* Backend capabilities go in plain language; *"Function names and tool schemas belong in the backend, which handles tool calls."*
- **What the prompt cannot do.** *"Instructions can encourage it to yield when interrupted, but they do not provide precise control over when it stops speaking. The model may keep speaking briefly after an interruption."* Required silence during tool calls and exact greetings are application jobs: *"Enforce required silence with application-controlled playback"*, and *"The model may paraphrase the greeting"* — the guide points to application-controlled playback *"when you need exact wording or verified delivery."* A prompt that relies on wording for any of these is a harness finding.

The delegation block of the guide's starter template (verbatim):

```text
Delegation policy:
Backend tools:
- [capability]: [what the backend can do]

Generate <|SpawnThinking|> when:
- The request needs a backend capability or careful reasoning.
- A correction changes the work already requested.

Do not generate <|SpawnThinking|> when:
- You can answer from the conversation or a still-current result.
- You need a brief clarification to understand the request.

Generate <|SpawnThinking|> before giving an answer that depends on backend work.
Do not guess the result while waiting.
```

Sources: developers.openai.com/api/docs/models/gpt-live-1.md, /api/docs/guides/live-prompting.md (read in full), /api/docs/guides/live-migration.md, /api/docs/changelog.md (GA 2026-09-10) — read 2026-10-31 (raw).

---

## GPT-5.6 — Sol / Terra / Luna (previous generation, 2026-07-09)

The generation before GPT-6, still callable in the API (not on the deprecations page, read 2026-10-31): **`gpt-5.6-sol`** (flagship), **`gpt-5.6-terra`** (strong at lower price), **`gpt-5.6-luna`** (efficient, high-volume); the bare alias `gpt-5.6` routes to Sol. It is also the API replacement OpenAI names for the retiring `gpt-5` / `o3` line. The July 9 launch date comes from press coverage — OpenAI's card doesn't carry one. Its predecessor GPT-5.5 is in `../archive/models/gpt.md`.

Wording-wise the three variants are one family: same guidance, differing capability and price. Treat "which of Sol / Terra / Luna" as a cost decision, not a prompting decision — with the usual caveat that the cheapest tier is least forgiving of vague scope.

### Headline behaviors

- **Leaner system prompts are now a measured win, not a taste call.** See family rule #1 — 10–15% better eval scores at 41–66% fewer tokens. On a 5.6 review, deleting duplicated instructions outranks rewriting them.
- **More concise by default than GPT-5.5** — the guide states it plainly. This inverts a common carried-over instruction; see below.
- **Reasoning effort gains `max`** — the ladder is `none` / `low` / `medium` / `high` / `xhigh` / `max`.
- **Autonomy boundaries are the guide's recommended framing** for agentic work (family rule #14).
- **Tone is specified as behavior, not as an adjective** (family rule #9).

### Verbosity inverted — the finding most likely to already be in the prompt

GPT-5.6 *"tends to be more concise by default than GPT-5.5."* A "be brief" / "keep it short" / "no preamble" line carried over from a 5.5-era prompt now **over**corrects: it stacks on an already-terse default and produces answers too thin to be useful. The fix is not to rewrite the instruction but to move the default out of prose entirely — set `text.verbosity` (`low`/`medium`/`high`) and leave only task-specific length rules in the body ("the summary section is at most 5 bullets").

**This is a cross-vendor trap, not just a GPT one.** The same review, targeting Claude Opus 5, needs the opposite edit: Opus 5's defaults run long, it does not calibrate to task, and `effort` won't shorten it — concision has to be prompted (`claude.md § Claude Opus 5`). One `AGENTS.md` cannot carry a single verbosity setting for both — and on GPT-6 Astra the GPT side flips too (detailed and formatted by default). When the prompt is cross-vendor, state length as a task requirement ("at most 5 bullets") rather than as a global disposition ("be concise" / "be thorough") — a requirement lands the same on both defaults.

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

### August 2026 update — the same name is now two post-trainings, split by surface

On **August 6, 2026** OpenAI updated **GPT-5.6 Sol in ChatGPT** (Plus / Pro chat) and made **GPT-5.6 Luna** the default for Free / Go users. The review-relevant fact: **"GPT-5.6" is no longer one model — the version depends on the surface.**

- **ChatGPT (Plus / Pro) = August Sol** — tuned for everyday conversation: more focused answers, tighter formatting, better fact-groundedness, an **effort slider** in ChatGPT for how much thought to put into a response. This August Sol "will only be available in the Chat experience in ChatGPT".
- **ChatGPT Free / Go = Luna (August)** — default model with unlimited text chats plus a **Think button** for higher reasoning on harder questions.
- **Work and Codex = July versions, unchanged** — OpenAI states explicitly: *"The version of GPT-5.6 Sol that powers Work and Codex is not changing as part of this release."* Codex / API reviews should target the **July** model behavior described above, not August.

**Review consequence:** when the artifact targets GPT-5.6, ask **which surface** — a ChatGPT-port tuned for the August chat model is not automatically valid for Codex/Work prompts (still July behavior), and vice versa. Wording guidance (family rules) is unchanged on both surfaces; the split is version identity, not wording.

---

## Universal / multi-version GPT prompts

A prompt that must work across two or more current GPT models (e.g., a Codex agent prompt that runs on `gpt-6.1-sol` *or* `gpt-6-luna` depending on the user's plan, or an API stack still split between 5.6 and GPT-6) is the GPT-side equivalent of a Claude universal prompt. Most production system prompts fall here.

### Rules

**1. Outcome-first and lean, for every version.**

Outcome-first phrasing is safe on the whole family. If **5.6** is in scope, also apply the leanness rule: a duplicated instruction is measured cost there (family rule #1).

**1a. Length and format defaults diverge inside the family.**

5.6 is terse out of the box; GPT-6 Astra tends toward detailed, formatted answers; the other GPT-6 models are undocumented on this axis. A global "be concise" or "be thorough" overshoots on one of them. Express length and structure as a task requirement ("at most 5 bullets", "one paragraph per finding", "rationale in prose") — that reads the same on every version.

**2. Don't lean on one model's parameters or window.**

- The effort ladder is per model: no `none` on Astra and 6.1 Sol, `none` on 6.1 Terra / 6 Sol / 6 Luna / 5.6 (§ GPT-6 family). Configure it per model, out-of-band; never write "answer without reasoning" into a shared prompt.
- Don't assume the 1M API window: 6.1 Sol defaults to 500K in Codex and ChatGPT Work.
- Don't assume the lowest hallucination rate — keep anti-hallucination snippets (`../techniques.md` §12, §22).
- Don't strip few-shot examples that are load-bearing on one model just because another prefers zero-shot. Keep them and test.

**3. Push output format into `json_schema` if the stack supports it.**

This is the most version-neutral lever. The system prompt becomes leaner regardless of which version reads it.

**4. Avoid hardcoding model names in the prompt.**

"You are GPT-6 Astra" locks you to the model. Prefer functional roles: "You are a code-review agent."

**5. Use `reasoning_effort` rather than wording for reasoning depth.**

Wording-based reasoning prompts ("think step by step") work inconsistently across versions. The API knob is consistent.

### Universal-GPT checklist

Before calling a prompt "universal across current GPT models", verify:

- [ ] No dependency on one model's context window (1M API vs 500K Codex default for 6.1 Sol) or hallucination rate
- [ ] No dependency on `reasoning_effort: none`, sampling parameters or Chat Completions tools unless every model in scope supports them
- [ ] Output contract is in `json_schema` if stack supports it; otherwise format is described once, not enforced through few-shot
- [ ] Reasoning depth is configured via `reasoning_effort`, not "think step by step" in prose
- [ ] Length and format stated per deliverable, not as a global disposition
- [ ] No process-step prescription the model would treat as noise (or it's tagged "step suggestions, not requirements")
- [ ] No model name hardcoded in the system prompt
- [ ] Tool-specific guidance is inside tool descriptions, not the system prompt
- [ ] Stable content first; volatile / user-specific content at the end

---

## Cross-vendor universal prompts (Claude + GPT)

The hardest case: an `AGENTS.md` or system prompt meant to work on **both** Claude Code (any current Claude) and Codex CLI (any current GPT model). The two vendors have **opposite defaults** on several axes; cross-vendor wording must be acceptable to both, which usually means picking the stricter constraint.

### The opposite-default table

| Axis | Claude default | GPT default (6.x / 5.6) | Cross-vendor wording |
|---|---|---|---|
| Process vs outcome | Tolerates step naming; literal — needs scope stated | Treats step prescription as noise | Outcome-first wording with steps as "typical sub-tasks, choose order yourself" |
| Few-shot examples | 3–5 diverse examples = standard | Often hurts reasoning models — zero-shot default | Skip few-shot unless format is genuinely strict; if needed, keep to 1–2 |
| Tool guidance location | System prompt is fine | Strongly prefer inside tool description | Move tool-specific guidance into tool descriptions; keep system prompt cross-tool only |
| Reasoning prompts | "Verify against criteria" works; "think harder" doesn't | `reasoning_effort` parameter; wording prompts mostly inert | Drop "think step by step" entirely; ask for specific verification or rely on the parameter |
| Persona / role | Functional one-line role useful | Outcome-first usually beats persona | Use functional role only when it encodes a methodological lens; otherwise drop |
| Subagent spawning | Fable 5.x / Opus 5.x delegate readily — need boundaries | GPT-6 Astra *"may delegate less often than desired"*; Codex spawns the subagents you defined | State delegation as a condition (when, how much), not a push in either direction |
| Output format | Prose constraints work | Push to `json_schema` when possible | Prefer `json_schema` for both; falls back to prose constraints if API doesn't support it |
| Aggressive emphasis | Overtriggers on every current Claude | Mostly inert / mild noise | Reserve ALL-CAPS / "CRITICAL:" for genuine safety invariants only |
| **Response length / format default** | **Opus 5 runs long and does not calibrate** — concision must be prompted, `effort` won't shorten it | **5.6 is already terse** — a carried-over "be brief" overcorrects; **GPT-6 Astra runs detailed and formatted** | State length as a **task requirement** ("at most 5 bullets"), never as a global disposition ("be concise" / "be thorough") |
| Repeated instructions | Tolerated; repetition near the end is even recommended on Opus 5 for concision | **Measured cost on 5.6** (leaner prompts: +10–15% score, −41–66% tokens) | State each rule once, in the section where it belongs; if a rule must be echoed, keep the echo to one short line |

### Rules for cross-vendor `AGENTS.md`

**1. Outcome-first wording, not step prescription.**

The worst case is the cross-vendor case for steps: if you write step-by-step, GPT reads it as noise; if you write pure outcome, a literal Claude may not fill in the scope the way you expect. The compromise: state the outcome first, then mention sub-steps as *typical* not *required*.

> Fix the failing CI pipeline. Typical sub-tasks: read the most recent failed log, identify the failing job, propose a fix, run tests locally before pushing. Adapt as needed.

This phrasing works on both sides because the outcome ("fix the failing CI pipeline") leads, and the steps are framed as guidance rather than constraint.

**2. Drop persona / role lines unless they're methodological anchors.**

"You are a senior engineer" adds attention noise on GPT and isn't load-bearing on Claude. Drop it. "Apply systems-thinking lenses: 2nd/3rd-order effects, 10-year horizon" is a methodological anchor — keep it on both sides.

**3. Watch the Codex 32 KiB cap.**

If the `AGENTS.md` is shared with Codex (i.e., it lives at the project root with no `.codex/` override), it's subject to Codex's `project_doc_max_bytes` (32 KiB by default). Claude Code has no equivalent hard cap but suffers context rot past ~300 lines. The strictest constraint wins: keep it short.

**4. Use markdown headers, not heavy XML, for structure.**

Both vendors handle markdown headers and XML well. Markdown is more readable in version control and travels better between tools that parse it (`agents.md` adopters, some IDE integrations). Reserve XML for inline structuring within sections, not as the primary skeleton.

**5. Tool descriptions over system prompt for tool guidance.**

This is OpenAI's strong recommendation; Anthropic tolerates either. Pick OpenAI's stricter rule for cross-vendor — it costs nothing on Claude and matters on GPT.

**6. Avoid model-name pinning.**

"When using Opus 5.5, do X" or "GPT-6 Astra should Y" inside an `AGENTS.md` defeats the purpose of cross-vendor. Functional descriptions only.

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

Mirror of the section in `claude.md`. These hold across GPT-6.x and GPT-5.6:

### Use `reasoning_effort` over wording for reasoning depth

`reasoning_effort` (ladder per model — § GPT-6 family) is the load-bearing knob. Wording-based reasoning prompts are inferior across the family. Migration rules differ per step: 5.5 → 5.6, test **one level lower** than the current setting; 5.6 → GPT-6, *"Preserve your current effective reasoning effort where supported"* (from `none` / `minimal` on Astra or 6.1 Sol, start at `low`); 5.6 Terra → 6.1 Terra, compare at the same effort.

### Prefer "verify" / "evaluate" / "consider" over "think"

Same as on Claude: "think harder" doesn't work; "verify your answer against these criteria before finishing" does.

### Aggressive emphasis is mostly inert noise

GPT doesn't overtrigger on "CRITICAL:" the way current Claude does, but it doesn't reliably elevate the rule's importance either. Use sparingly; rely on structure (XML, headers) and outcome wording instead.

### Prompts age fast

OpenAI ships frequent minor versions and the prompting style shifts more between them than Claude generations do. A prompt that's 6 months old should get a fresh-eyes review on every model upgrade — don't assume the old wording still pulls its weight.
