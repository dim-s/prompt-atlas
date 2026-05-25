# Core wording techniques

Each technique covers WHEN it applies and the concrete phrasing. Keep it tight — this is a reference you load when you need a specific tool, not a textbook to read front-to-back.

---

## 1. Clarity and directness

**Rule**: Say what you want, specifically.

> Golden rule: show your prompt to a colleague who lacks context on the task. If they'd be confused, Claude will be too.

| Weak | Strong |
|---|---|
| "Create an analytics dashboard." | "Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation." |
| "Add tests for foo.py." | "Write a test for foo.py covering the edge case where the user is logged out. Avoid mocks." |
| "Fix the bug." | "Users report login fails after session timeout. Check the auth flow in src/auth/, especially token refresh. Write a failing test that reproduces the issue, then fix it." |

**Common miss**: assuming Claude shares your context. It doesn't. Include files, scenarios, edge cases, success criteria.

---

## 2. Explain the WHY

**Rule**: Don't just state rules — explain the reason. Claude generalizes from the explanation.

| Weak | Strong |
|---|---|
| "NEVER use ellipses." | "Your response will be read aloud by a text-to-speech engine, so never use ellipses since the engine will not know how to pronounce them." |
| "Use snake_case." | "Use snake_case for all Python identifiers — variables, functions, method names, and modules — to match PEP 8 and the rest of this codebase." |
| "Keep responses short." | "Users read these responses on mobile between tasks, so keep them short: one paragraph, two max." |

**Why this works**: when Claude understands the reason for a rule, it can handle edge cases the rule didn't anticipate. When Claude only knows the rule, it applies it mechanically and breaks on the first case you didn't think of.

---

## 3. Positive framing over negative

**Rule**: Tell Claude what TO do, not what NOT to do.

| Weak | Strong |
|---|---|
| "Do not use markdown." | "Respond in smoothly flowing prose paragraphs." |
| "Don't start with 'Here is...'." | "Respond directly with the content itself." |
| "Never make up function names." | "Only reference functions you have verified exist in the code." |

Negative instructions leave Claude guessing what to do instead. Positive instructions give it a target.

**When negation is genuinely needed**, pair it with the positive alternative:
> "Never use `eval()`. Use `JSON.parse()` for JSON and `ast.literal_eval` for Python literals."

---

## 4. XML structure for multi-part prompts

**Rule**: wrap each logical section in its own XML tag when the prompt mixes instructions, context, examples, and inputs.

Claude is trained to parse XML as structure. Tags reduce misinterpretation.

```xml
<role>You are a senior security engineer.</role>

<instructions>
Review the code below and identify security issues.
</instructions>

<code>
{{code_content}}
</code>

<output_format>
Return a list of issues, each with: severity, location, one-sentence description, suggested fix.
</output_format>
```

**Best practices:**

- Descriptive tag names (`<code>` not `<input_1>`).
- Consistent across your prompts so the same meaning maps to the same tag.
- Markdown headers (`## Instructions`, `## Context`) work too — Anthropic's Context Engineering guidance treats them as interchangeable with XML for delineation.
- Nest when content has natural hierarchy: `<documents><document index="1"><source>foo.pdf</source><document_content>...</document_content></document></documents>`.

**When to use**: prompts with multiple clearly distinct parts, OR any prompt longer than ~200 words, OR when different content types need to be distinguishable.

**When to skip**: short simple prompts where XML just adds tokens without adding clarity. Modern Claude models handle plain prose well. Don't wrap a three-sentence instruction in five nested tags.

---

## 5. Examples (few-shot)

**Rule**: 3–5 examples, wrapped in `<example>` tags, diverse enough to prevent overfitting.

```xml
<examples>
  <example>
    <input>Added user authentication with JWT tokens</input>
    <output>feat(auth): implement JWT-based authentication</output>
  </example>
  <example>
    <input>Fixed race condition in payment processor</input>
    <output>fix(payments): resolve race condition in processor</output>
  </example>
  <example>
    <input>Updated docs to reflect new API shape</input>
    <output>docs(api): update for new response shape</output>
  </example>
</examples>
```

**Requirements**:

- **Relevant** — mirror the actual use case.
- **Diverse** — cover edge cases. If all examples start with the same word, Claude learns that as a pattern.
- **Structured** — wrap in consistent tags.

**Sequencing**: start with 1 example, add more only if output still drifts from what you want.

**Anti-pattern**: examples that are all too similar — Claude extracts accidental patterns you didn't intend.

---

## 6. Role prompting

**Rule**: set a role when the task is generative, advisory, or alignment-dependent. Keep it minimal or skip when the task is factual retrieval or discriminative.

> "You are a senior backend engineer specializing in distributed systems."

A role shapes: tone, default depth of explanation, which details the model surfaces, how it handles uncertainty.

### Task-type gating (research-grounded)

The effect of role/persona framing is NOT uniform — it's strongly task-dependent. Recent research (USC 2026, [arxiv 2603.18507](https://arxiv.org/abs/2603.18507) — "Expert Personas Improve LLM Alignment but Damage Accuracy") quantified the split:

| Task type | Persona/role effect |
|---|---|
| **Generative, alignment-dependent** (writing, advice, strategy, role-play, safety, brainstorming, therapy-adjacent work) | **Helps.** Role framing activates the right tone, stance, and methodological lens. Specific methodological anchors (time horizons, named frameworks) improve output further. Safety personas lifted attack-refusal by +17.7 p.p. in the USC study. |
| **Factual / discriminative** (math, coding, MMLU-style recall, extraction) | **Hurts.** "You are an expert X" activates instruction-following mode at the cost of factual recall. MMLU accuracy dropped 71.6% → 68.0% with expert personas in the same study. |

**Practical implication**: a strategy-advice, creative-writing, or therapy-adjacent prompt benefits from a role + methodological frame. A code-completion or factual-lookup prompt typically does not — keep the role line minimal or drop it entirely.

### Generic persona vs. methodological anchor

This is the distinction that prompt-tuning often misses. A role block can look superficially similar but fall into either bucket.

**Generic persona (neutral-to-negative, especially on factual tasks):**
> "You are an expert full-stack developer."

No new facts added. Mechanism (per the research): "persona prefixes activate the model's instruction-following mode that would otherwise be devoted to factual recall."

**Methodological anchor (positive, especially on generative/advisory tasks):**
> "You are a senior strategy advisor. Apply these lenses: 2nd/3rd-order effects analysis; 10–30 year horizon; behavioral economics; Daoist non-action (У-Вэй) where forcing action is costly."

This names the *frames* to apply, not just the *label* of expertise. On literal models (Opus 4.7 especially), these anchors are load-bearing — the model will NOT spontaneously apply "2nd-order effects" or a "10-year horizon" unless named explicitly.

**Rule of thumb when reviewing a competency/role block**: for each item, ask whether it names a **specific frame, horizon, method, or stance** the model can literally apply (keep) or merely asserts **expertise level** (drop). Don't confuse specificity-that-looks-abstract with genuine vagueness.

Examples:
- ✅ "Systems synthesis (2nd/3rd-order effects)" — names the lens and its depth
- ✅ "10–30 year strategic horizon" — measurable time scope
- ✅ "Objective mirror without cognitive biases" — stance; pairs with Zero-Flattery rule
- ❌ "Strategic thinking expertise" — generic label
- ❌ "Deep analytical capabilities" — unmeasurable vibe

### Keep it functional

"You are Dr. Emily Chen, 42, Stanford grad, 15 years at FAANG..." adds nothing behavioral — just decoration. Biographical detail ≠ methodological anchor. Strip biography; keep frames.

### When to skip roles entirely

- Very short focused tasks (classification, extraction, one-shot formatting)
- Pure factual lookups where persona framing risks interfering with recall
- Code tasks where correct code matters more than "senior engineer vibe"

Anthropic's 2026 guidance notes that on modern models "heavy-handed role assignment" is often unnecessary for factual/coding tasks — being explicit about the desired *perspective* ("focusing on thread safety", "from a security-first point of view") often works as well as naming a persona. See also [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) on the "right altitude" principle.

### When roles help most

- Tone and register matter (customer-facing assistants, pedagogy, therapy-adjacent)
- The assistant should take a specific stance (skeptical reviewer vs supportive mentor)
- You want to activate particular methodological lenses (security-first, systems-thinking, user-empathy)
- Strategic or creative work where the frame IS the value

---

## 7. Match prompt style to desired output style

**Rule**: the markdown density of your prompt leaks into the output.

If you want plain flowing prose, write the prompt in plain flowing prose. If you want bulleted output, write in bullets. If you want code-heavy terse output, keep the prompt code-heavy and terse.

This is one of the easiest wins. People who can't get Claude to stop producing markdown bullets often have markdown-bullet-heavy prompts.

---

## 8. Long-context structure

**Rule**: long data at the TOP, the question at the BOTTOM. Wrap documents in XML.

```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>
      {{ANNUAL_REPORT}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis_q2.xlsx</source>
    <document_content>
      {{COMPETITOR_ANALYSIS}}
    </document_content>
  </document>
</documents>

Analyze the annual report and competitor analysis. Identify strategic advantages and recommend Q3 focus areas.
```

Queries at the end improve response quality meaningfully on complex multi-document inputs.

**Quote-grounding variant** — have Claude extract relevant quotes FIRST, then answer. Reduces hallucination on long documents:

> "Find quotes from the documents relevant to the question. Place them in `<quotes>` tags. Then, based only on those quotes, answer the question. Place your answer in `<answer>` tags."

---

## 9. Action mode vs suggestion mode

**Rule**: the verb you use determines whether Claude acts or describes.

| Suggestion verbs (Claude describes) | Action verbs (Claude does) |
|---|---|
| "Can you suggest changes to..." | "Change..." |
| "What would you improve in..." | "Improve..." |
| "How should we refactor..." | "Refactor..." |

If you want action, use action verbs. If you want options, use suggestion verbs.

**Proactive-action snippet** (agentic system prompt):

> `<default_to_action>`
> By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing.
> `</default_to_action>`

**Conservative-action snippet** (opposite bias):

> `<do_not_act_before_instructions>`
> Do not jump into implementation unless clearly instructed. When intent is ambiguous, default to providing information, doing research, and making recommendations. Only proceed with edits when the user explicitly requests them.
> `</do_not_act_before_instructions>`

---

## 10. Emphasis (caps, IMPORTANT:, YOU MUST)

**Rule**: use sparingly. Emphasis that's everywhere is emphasis nowhere.

Rough guidance:

- **Safe to use**: 1–2 emphasized rules per document, reserved for genuine invariants (secrets, data loss, destructive ops).
- **Overused**: every other bullet starts with "IMPORTANT:" or "YOU MUST" or has ALL CAPS segments.
- **On newer models (4.5+)**: overuse actively causes overtriggering — the model applies the rule beyond its intended scope.

Good emphasis:

> **IMPORTANT**: Never commit files matching `*.env` or `secrets.*` — they contain credentials.

Bad emphasis:

> **IMPORTANT**: Write clean code.
> **YOU MUST**: Follow conventions.
> **CRITICAL**: Use good variable names.

These don't help — they just dilute attention.

---

## 11. Minimize overengineering (agentic coding)

Newer Claude models tend to overbuild: extra files, unnecessary abstractions, flexibility nobody asked for. Counter with an explicit scope-limit snippet:

```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:
- Scope: don't add features, refactor, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up.
- Documentation: don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-evident.
- Defensive coding: don't add error handling or validation for scenarios that can't happen. Trust internal code and framework guarantees.
- Abstractions: don't create helpers or utilities for one-time operations. Don't design for hypothetical future requirements.
```

---

## 12. Anti-hallucination for code tasks

```
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Investigate and read relevant files BEFORE answering questions about the codebase. Never make claims about code before investigating, unless you are certain of the correct answer.
</investigate_before_answering>
```

---

## 13. Anti-hard-coding / anti-test-gaming

```
Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them.
```

---

## 14. Cleanup-after-yourself

```
If you create any temporary files, scripts, or helper files for iteration, clean them up at the end of the task.
```

---

## 15. Safety / reversibility for agents

```
Consider the reversibility and potential impact of your actions. Local, reversible actions (editing files, running tests) are fine without confirmation. For actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.

Actions that warrant confirmation:
- Destructive: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse: git push --force, git reset --hard, amending published commits
- Externally visible: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. Don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be in-progress work.
```

**When to use**: any agent with tool access that can affect shared state.

---

## 16. Long-horizon / multi-window work

Context-awareness prompt (for agents expected to save state and continue across sessions):

```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your limit, save current progress and state to memory before the window refreshes. Always be as persistent and autonomous as possible; complete tasks fully even as the budget fills. Never artificially stop early.
```

State-tracking prompt:

```
This is a long task. Maintain state using:
- A structured progress file (progress.json) for completed/pending work
- An unstructured notes file (progress.txt) for free-form context
- Git for checkpointing

Before starting, review progress.json, progress.txt, and recent git log. Focus on incremental progress: complete one item fully before starting the next.
```

---

## 17. Subagent orchestration

When to spawn subagents (for a main agent):

```
Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't share state. For simple tasks, sequential operations, single-file edits, or tasks needing shared context across steps, work directly rather than delegating.
```

When to encourage subagent usage specifically on Opus 4.7 (which spawns fewer by default):

```
Do not spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see). Spawn multiple subagents in the same turn when fanning out across items or reading multiple files.
```

---

## 18. Research and information-gathering prompt

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down the research task systematically.
```

---

## 19. Verbosity control

Reduce verbosity:

```
Provide concise, focused responses. Skip non-essential context, and keep examples minimal.
```

Reduce markdown density (useful when you want flowing prose):

```
<avoid_excessive_markdown_and_bullet_points>
When writing reports, documents, technical explanations, analyses, or any long-form content, write in clear, flowing prose using complete paragraphs and sentences. Reserve markdown primarily for inline code, code blocks, and simple headings.

Do not use ordered or unordered lists unless: (a) you're presenting truly discrete items where a list is the best format, or (b) the user explicitly requests a list.

Instead of listing items with bullets or numbers, incorporate them naturally into sentences. Your goal is readable, flowing text that guides the reader naturally through ideas rather than fragmenting information into isolated points.
</avoid_excessive_markdown_and_bullet_points>
```

Increase verbosity / get summaries:

```
After completing a task that involves tool use, provide a quick summary of the work you've done.
```

---

## 20. Self-check / verification

```
Before finishing, verify your answer against these criteria:
1. [criterion]
2. [criterion]
3. [criterion]

If any criterion fails, revise before returning your answer.
```

Reliably catches errors especially in coding and math work. More effective than "think carefully" or "double-check".

---

## 21. The golden rule

When in doubt, apply the colleague test:

> If I showed this prompt to a smart colleague who doesn't know the context of my project, would they be able to follow it unambiguously?

If the answer is no, add specifics. If they'd ask "what does X mean?", answer that question in the prompt. If they'd ask "why?", add the WHY.

---

## 22. Give Claude permission to express uncertainty

**Rule**: explicitly allow the model to say "I don't know" or acknowledge limitations rather than forcing it to guess. Anthropic's 2026 guidance cites this as one of the most reliable ways to reduce hallucinations.

Snippet:

> If you're not sure about something — a file, a function name, a fact, an edge case — say so clearly rather than guessing. "I don't have enough information to answer this with confidence" is a valid and preferred response over a plausible-sounding fabrication.

For agentic coding specifically, pair with:

> If a required tool isn't available, or an API/file you need doesn't exist, stop and ask rather than inventing a substitute.

**Why it works**: without explicit permission, the model implicitly optimizes for *sounding confident and useful*, which pushes it toward guesses. Permission to be uncertain lets the model's actual calibration show through.

**When to use**: any prompt where factual accuracy matters — code that references specific APIs, data analysis with specific numbers, documentation, any output the user will take as authoritative.

---

## 23. Start simple, add complexity only when you've seen a failure

**Rule**: don't pre-add scaffolding for problems you haven't observed. Every line of prompt is attention spent.

The Anthropic 2026 guidance is explicit: *"The best prompt isn't the longest or most complex. It's the one that achieves your goals reliably with the minimum necessary structure."*

Method:
1. Write the minimum viable prompt — role + task + format.
2. Run it on 3-5 realistic inputs.
3. Note specific failure modes.
4. Add ONE change that addresses a specific observed failure.
5. Re-test. If the failure is gone, keep the change; if not, try a different angle or remove it.

**Anti-pattern**: copy-pasting a 2000-word "mega prompt" template as a starting point. Most of it won't be earning its keep, and the important parts get buried.

---

## 24. Chain-of-thought levels

Three progressively structured ways to elicit reasoning when thinking isn't enabled:

**Basic** (one line):
> "Think step by step before answering."

**Guided** (named reasoning stages):
> "Before answering, first identify the edge cases, then list the constraints, then evaluate each candidate solution against the constraints."

**Structured** (XML tags separating reasoning from answer):
> "First, in `<thinking>` tags, work through the problem. Then, in `<answer>` tags, give your final answer."

Pick the least structure that gets good results. On Opus 4.7 and Sonnet 4.6 with adaptive thinking available, the model often handles decomposition better internally than any hand-written CoT plan. Use manual CoT when you want *visible* reasoning (for debugging prompts, auditing behavior, or teaching).

---

## 25. Unlock creativity, boldness, and "додумывание"

**Rule**: literal models (Opus 4.7 especially) won't propose bold alternatives, push back on bad framing, or generate options beyond the first obvious one — unless you explicitly permit and request it. Anthropic's own guidance: *"the model will not silently generalize an instruction from one item to another, and will not infer requests you didn't make."* That includes creativity — it's also a request the model won't infer.

There are two distinct levers, and the strongest prompts combine both:
- **Universal unlockers** (this technique) — give the model permission and form of free thinking. Independent of domain.
- **Project anchors** (technique 6 — role/methodological anchors) — give the model direction. Domain-specific.

Universal unlockers without anchors → creative but unfocused. Anchors without unlockers → focused but literal and timid. Combine them.

### The five universal unlockers

Pick the 1–2 that match the failure mode you're seeing. Don't stack all five — that's noise.

**a) Force divergence — "N options before a choice"** (the single highest-leverage pattern):

> Before proposing a solution: generate 3 distinct options, each with a one-line rationale and the main trade-off. If none feel strong, say so and ask. If one is clearly best, state why — don't pad the list.

Use when the model converges on the first obvious answer. Anthropic recommends this pattern explicitly for visual-design tasks on Opus 4.7; it generalizes to code, text, strategy, product decisions. The "if one is clearly best" clause prevents fake diversity.

**b) Permission to push back** — counters "obedient executor" mode:

> If the task as stated looks like it optimizes for the wrong thing, or if there's a materially better approach than what was asked — surface it before executing. A 30-second pushback is cheaper than a finished solution in the wrong direction.

Use when the user notices the model executes bad framings without question. Especially valuable for advisory, strategy, or design tasks where the framing IS part of what the model should evaluate.

**c) Divergent → convergent phases** — separates generation from filtering:

> Phase 1 — brainstorm without filtering: 5–7 options, including ones you think are wrong. Phase 2 — rank them and recommend one. Keep both phases visible in the response.

Use when the model self-filters too early and only shows safe choices. The explicit "including ones you think are wrong" clause is the active ingredient — without it, the model filters silently during phase 1.

**d) Steelman the counter-case**:

> Before finalizing, argue the strongest case AGAINST your proposal. If the counter-case is stronger than you expected, revise.

Use when proposals feel one-sided or the model doesn't surface risks. Especially useful pre-commit for plan-mode outputs and architectural proposals.

**e) First principles / reframe**:

> Before solving: state the underlying problem in one sentence, as if the current framing didn't exist. Then check whether the proposed solution actually addresses it.

Use when the model gets stuck polishing a solution to the wrong problem.

### Permission to express taste-uncertainty

Pair any of the above with a taste-level permission clause — otherwise the literal model picks silently on matters of taste:

> If 2–3 options land equally well on the success criteria and the difference is taste — present them as a list with trade-offs and wait for a choice. Don't pick silently.

### Non-prompt lever — surface before rewriting

On Claude Code with Opus 4.7, `effort: low`/`medium` strictly scopes work to what was asked ("rather than going above and beyond" — Anthropic migration guide). If the user reports "the model isn't creative enough", check effort first. `high` or `xhigh` restores much of the "додумывание" without any prompt change. No prompt technique overcomes low effort on 4.7.

### Why these work

Without explicit permission, the model optimizes for *sounding confident and proceeding directly*. That bias suppresses: (1) alternatives to the first idea, (2) pushback on the framing, (3) admission of uncertainty, (4) ideas the model predicts the user won't like. Each unlocker above neutralizes one of those suppressors.

**When NOT to use**: highly-constrained executional tasks (structured extraction, known-format conversion, deterministic code fixes). Unlockers add noise when there's genuinely one right answer.

### Whole-agent creative kernel — install once at system-prompt level

The unlockers above are *per-task*: you add them at the moment you need divergence. When the *entire agent* lives in a creative / advisory / narrative / non-technical domain (game design, narrative, content, brand strategy, coaching, brainstorming partner, design crit, fiction editor), per-task unlockers force the user to retype them every turn. Install a **creative-domain kernel** once at the system-prompt level instead.

Apply to: agent system prompts (CLAUDE.md role section, subagent body, role-system text) where the agent's permanent function is generative/advisory rather than executional.

#### Detection — when prompt-atlas should propose this

The agent's prompt centers on at least two of these:
- Generating ideas, options, or alternatives as primary output
- Critiquing, evaluating, or counseling the user on subjective choices
- Producing narrative / character / lore / copy / pitch / scenario material
- Strategic / product / design decisions where framing IS the value
- Named domain: game design, narrative design, screenwriting, content, brand, marketing, strategy, coaching, therapy-adjacent, brainstorming, advisory, design

User symptoms that point here even without an explicit "creative" label:
- "feels mechanical / robotic / like a checklist"
- "doesn't push back, just executes"
- "won't suggest things I didn't ask for"
- "lost the warmth / personality"
- "gives one obvious answer instead of options"
- "skips straight to bullet points when I want prose"
- Russian-domain cues: «сухо», «скучно», «без огонька», «не додумывает», «выдаёт буллеты», «слишком технично», «как QA, а не как дизайнер»

#### The kernel — composable building blocks

Don't paste all of these at once. Pick the 3–6 that match the agent's domain. Each line overrides one specific Opus 4.7 default that conflicts with creative work. Source for each override is Anthropic's own [Opus 4.7 migration guide](https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7) or the leaked 4.7 chat system prompt.

**Role re-frame** — overrides "obedient executor" default:
> You are a senior collaborator in this domain, not a junior executor. Your job is to understand what I'm trying to do, not to literally execute what I wrote. If the better move differs from what I asked, say so before doing what I asked.

**Expansion license** — overrides "no inference" default. Anthropic's migration guide flags loss-of-inference as the #1 4.7 creative regression: *"if you've been counting on Claude to read between the lines… the output will feel flatter."*
> Treat my request as the minimum scope, not the maximum. If you see adjacent moves I missed, raise them and mark them "кстати: …" / "bonus: …". If you think I'm asking the wrong question, say so before answering.

**Tone re-frame** — overrides 4.7's flatter, less validation-forward default. Anthropic itself reinstalls warmth in its chat system prompt for 4.7 (*"Claude uses a warm tone"*) — i.e. they didn't trust the weights.
> Warm, direct, curious. Prose by default — no headers, bullets, or emoji unless I ask. No "Great question!", no apologies for length, no summary at the end, no restating my question back to me.

**Ambiguity handling** — overrides 4.7's checklist-clarify drift:
> If something is ambiguous, voice the most likely interpretation and proceed. Don't stop for a list of clarifying questions unless the answers genuinely block the work.

**Default divergence** — overrides "first obvious answer" convergence. Anthropic recommends this pattern explicitly for visual-design tasks on Opus 4.7; it generalizes to all creative work.
> When I ask for options or ideas: minimum 3 from genuinely different roots, not variations of one. Include one you'd consider risky or wrong. Don't pick a winner — that's my job.

**Default pushback** — overrides validation-forward mode for advisory/critique agents:
> Before agreeing with my proposal, steelman the opposition in one sentence. Only then give your position. Don't validate an idea just because it's mine.

**Reasoning depth trigger** — works with adaptive thinking; Anthropic confirms this language shifts depth.
> For non-trivial questions: think carefully step-by-step, this is harder than it looks. For simple ones: answer directly.

**Anti-defaults block** — name the unwanted behaviors out loud. Positive examples beat prohibitions, but a short prohibition list still helps for habitual patterns and is cheaper than re-training each session.
> Don't: open with "I'd be happy to", hedge with "depending on context", paste a TL;DR at the start of a long answer, structure prose answers as bullet lists.

#### Show, don't tell — the style anchor

The single highest-leverage line for creative work on 4.7 is a positive style example. Recommend that the user paste 1–3 short paragraphs in the target register — the kernel's tone instructions become 10× more effective when paired with a concrete distribution to imitate.

> Style anchor — write in this register:
> [user pastes their own paragraph or a paragraph from a reference author]

Without this, "warm, direct, prose" is adjectives the model has to interpret. With it, the model has a reference example to mirror — this is the same lever Anthropic recommends throughout its prompting docs ("positive examples beat negative rules").

#### Effort-level nuance for creative work

Anthropic's general advice on 4.7 is "raise effort for stronger output". For creative agents this needs a refinement — `xhigh` makes the model self-edit harder, which can prune exactly the weird/risky options that brainstorming wants. Heuristic:

| Effort | Use for creative agents |
|---|---|
| `xhigh` | Structured creative work with a frame: GDD section, balance design, plot outline against given constraints, strategy memo, brand voice doc. Quality > quantity. |
| `high` | Default for most creative work. Good balance of depth and weirdness retention. |
| `medium` | Pure divergent brainstorm, raw ideation, "give me 20 wild options". Lets weirder branches survive. |
| `low` | Almost never for creative agents — produces shallow output. |

Surface this as a non-prompt lever before proposing kernel edits, same as elsewhere on 4.7.

#### Why the kernel works as a whole

Opus 4.7's defaults are tuned for code, agentic execution, and instruction precision. The kernel overrides each default that conflicts with creative work: literalism → expansion license; flatness / less validation → tone block; obedient executor → collaborator + pushback; first-obvious-answer → mandatory divergence; bullet-format drift → prose default; checklist-clarify → "interpret and proceed". Each line targets a documented behavior shift between 4.6 and 4.7. The kernel is a one-time install for a permanent domain, not a per-task addendum.

#### When NOT to use

- **Mostly technical agent that occasionally does creative work** — use per-task unlockers (§25 above) instead; the tone-kernel will pollute regular output.
- **Agent has a strong existing domain frame with conflicting register** (e.g. formal legal advisor, compliance bot) — keep the divergence + license clauses, adapt or drop the tone line.
- **Cross-vendor universal prompt** — the tone block is Claude-shaped (warm-by-default needs explicit installation specifically on 4.7); Gemini 3 and GPT-5.5 react differently. For cross-vendor, drop the tone block and keep only role re-frame + expansion license + divergence — those generalize.
- **Sonnet 4.6 / Haiku 4.5 targets** — Sonnet is less literal than 4.7, so the kernel is helpful but lighter touch suffices; Haiku struggles with the more abstract clauses, prefer concrete examples.

---

## 26. Don't over-trust visible reasoning

**Research caveat**: Anthropic's Alignment Science team found that Claude's visible chain-of-thought mentions the actual influences on its reasoning only ~25% of the time on average, and 41% of the time even for ethically loaded hints. In reward-hacking scenarios, models exploited hints 99% of the time while verbalizing that behavior <2% of the time.

**What this means for prompt design**:
- Don't assume that reviewing the displayed reasoning will catch all bugs in how the model is interpreting your prompt.
- If you're debugging a prompt by reading visible CoT, also check input/output behavior on held-out cases — the reasoning might be a post-hoc rationalization.
- "Explain your reasoning" in the output helps communication with the user but isn't a reliable model-behavior audit trail.

You don't need to do anything special in the prompt for this — just don't over-index on visible reasoning as ground truth.

---

## 27. Outcome-first framing (vendor-dependent — strongest on GPT-5.x)

**Rule**: lead with the target outcome and success criteria. Mention sub-steps as guidance, not requirement.

OpenAI's GPT-5.5 prompting guide is unusually direct: *"GPT-5.5 treats detailed step-by-step instructions as interference: redundant instructions create noise, narrow the solution space, and make responses overly mechanical."* The same family-wide preference holds for 5.4 and 5.3, weaker but real. On the Claude side, Opus 4.7's literalism makes step naming useful when scope must be explicit — but step prescriptions framed as *requirements* still degrade output quality where there's a better path the model can find on its own.

| Process-prescription (often weak) | Outcome-first (often strong) |
|---|---|
| "First read the file. Then identify the bug. Then propose a fix. Then write a test. Then implement the fix." | "Fix the bug in `src/auth/session.py` that causes logout to fail after token refresh. Verification: existing tests pass plus a new regression test you add." |
| "Step 1: validate input. Step 2: query database. Step 3: format response." | "Endpoint must return 200 with the matching record on valid input, 404 on missing record, 400 on malformed input. Choose the implementation order yourself." |
| "Apply the migration in this order: backup → drop column → rebuild index → run smoke test." | "Migration completes without data loss and `migrate_test.sh` passes. Backup before any destructive step." (the steps the model takes are its problem; the safety invariant is explicit) |

**Cross-vendor compromise**: when targeting both Claude and GPT-5.x, state the outcome first then mention sub-tasks as *typical, not required*:

> Fix the failing CI pipeline. Typical sub-tasks: read the most recent failed log, identify the failing job, propose a fix, run tests locally before pushing. Adapt as needed.

**When NOT to apply**:
- Claude Opus 4.7 prompts where you've observed the model skipping a step that matters — explicit naming wins there. The Prime directive applies (don't strip a step that was added in response to a real failure).
- Prompts where the order is genuinely load-bearing (e.g., "lock the file BEFORE reading it" — naming the order is the point of the rule).

---

## 28. Output contract: API parameter, not prompt prose

**Rule**: when the output must conform to a specific shape, push the contract into the API parameters (`json_schema` with `strict: true`, response format constraints, structured outputs) rather than describing the shape in the system prompt.

**Why it matters**:
- Prompt-prose contracts compete with the rest of the system prompt for attention. The model sees "Return JSON with fields X, Y, Z…" alongside everything else, and may still drift on edge cases.
- API-level contracts are enforced by the decoder, not the model's interpretation. Strict mode either succeeds or fails — there's no "mostly correct" failure mode.
- The system prompt becomes leaner and easier to read. Behavior stays in the prompt; format moves to the parameter.

**Weak (format in prose):**

> "Return your answer as JSON with these fields: `name` (string), `age` (number), `tags` (array of strings). Make sure age is between 0 and 150. Do not include other fields. Do not wrap the JSON in markdown."

**Strong (format in API parameter):**

System prompt:
> "Extract person facts from the input."

API call:
```json
{
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "person",
      "schema": { "type": "object", "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer", "minimum": 0, "maximum": 150},
        "tags": {"type": "array", "items": {"type": "string"}}
      }, "required": ["name", "age", "tags"], "additionalProperties": false },
      "strict": true
    }
  }
}
```

**Cross-vendor**: both Anthropic (Claude API) and OpenAI (Responses API) support structured outputs / `json_schema`. The wording fix is the same on both: move the format out of the prompt.

**When the stack doesn't support `json_schema`**: keep a tight prose contract — but only one. Don't repeat the format in the system prompt AND in few-shot examples AND in a final reminder. Pick one location, name the format precisely, move on.

**This skill is about wording.** Pointing the user at the right API parameter is part of the review even if you can't change the API call yourself — flag it as: "Move this format spec out of the prompt and into `json_schema`. The system prompt then drops the format paragraphs."

---

## 29. Few-shot calibration for reasoning models

**Rule**: on reasoning-heavy tasks with modern models, fewer examples is often better. Sometimes zero-shot beats few-shot.

OpenAI's reasoning best-practices: *"Zero-shot examples initially, few-shot only if needed."* And: *"With today's reasoning models, clear instructions and well-defined constraints often work better than adding examples. Research shows few-shot prompts can reduce performance when the task requires heavy reasoning."*

This is a major shift from the GPT-3 / Claude-2 era, where 3–5 examples was the default. The reasoning trace generated by modern models can be derailed by examples that pattern-match too narrowly.

### Decision tree for few-shot

1. **Try zero-shot first.** Crisp task description, success criteria, output contract via `json_schema` if applicable.
2. **If output drifts on format only**: add 1–2 examples covering the strict format. Don't add more — diminishing returns and rising risk of pattern overfitting.
3. **If output drifts on content / reasoning quality**: examples likely won't fix it. Raise `reasoning_effort` (GPT-5.x) or move to a stronger model. Don't paper over reasoning issues with example stacks.
4. **If targeting Claude (any current version)**: keep 3–5 examples as a default for format-steering tasks where format matters more than reasoning. The Claude family is more example-tolerant than GPT-5.x.

### Anti-pattern: 5+ examples on a reasoning-heavy GPT-5.x prompt

Five "demonstration" examples consume both attention and tokens, and on GPT-5.5 they often hurt the very thing the user wants — careful reasoning. Replace with one terse success-criteria spec.

### When few-shot still earns its keep on GPT-5.x

- Output format is unusually strict and `json_schema` isn't an option (e.g., domain-specific markup, custom DSL).
- The format requires implicit conventions the user can't easily articulate ("write in our team's voice"). Show, don't tell.
- 1–2 examples; never a wall.

---

## 30. Reasoning depth via parameter, not prose (`reasoning_effort` / `effort`)

**Rule**: control reasoning depth with the API knob, not prompt wording.

Both vendors expose a reasoning-depth parameter:
- **OpenAI GPT-5.x**: `reasoning_effort` with values `none` / `low` / `medium` / `high` / `xhigh`.
- **Anthropic Claude 4.x**: `effort` with values `low` / `medium` / `high` / `xhigh`.

Wording-based reasoning prompts ("think step by step", "think harder") are weak substitutes. They sometimes work on older models but on the current generations they're inert at best, counterproductive at worst.

### When you see "think step by step" in a prompt, the wording fix is

1. Ask the user what `reasoning_effort` (or `effort`) they're running at.
2. If `low` or `none`, propose raising it before changing anything in the prompt.
3. If already `high` or `xhigh` and reasoning is still shallow, the model probably can't do better — escalate to a more capable model rather than rewriting the prompt.

### Wording prompts that DO help reasoning depth

These are not "think harder" — they're requests for *specific kinds of reasoning*:

- "Verify your answer against these criteria before finishing: [criterion 1], [criterion 2]…"
- "First identify the hardest sub-problem, then solve it before the easier parts."
- "List the cases where your reasoning could be wrong, then address each."
- "Before finalizing, argue the strongest counter-case to your proposal. If it's stronger than expected, revise."

These compose with the parameter — they shape *how* the model uses its reasoning budget, while the parameter sets *how much* budget exists.

---

## 31. Preambles for tool-heavy agentic flows (GPT-5.x / Codex)

**Rule**: in long agentic traces with multiple tool calls, instruct the model to emit a short preamble before each phase.

OpenAI's official guidance: *"a brief visible update that acknowledges the request and states the first step. Keep it to one or two sentences."* In the Responses API, this is encoded with `phase: "commentary"` for intermediate updates and `phase: "final_answer"` for the completed result — but the wording fix is the same regardless of API.

Snippet:

> Before calling a tool, emit a one-sentence preamble that names the tool and what you expect to learn or change. After each major phase (exploration / implementation / verification), emit a 2-3 sentence status update. Reserve the final paragraph for the completed answer.

**When to use**:
- Codex CLI agents performing long multi-tool workflows where the user wants visible progress.
- Production agents on the Responses API where downstream consumers separate `phase: "commentary"` from `phase: "final_answer"`.

**When NOT to use**:
- Single-turn or short-turn interactive prompts — the preamble is overhead.
- Claude prompts on Opus 4.7 — the model already calibrates progress updates well; explicit preamble instructions are redundant (and listed as anti-pattern #7 in `antipatterns.md`).

**Failure mode if you skip this on a tool-heavy GPT-5.x prompt**: streaming UI feels silent until the final tool returns; users assume the agent stalled. The wording fix restores responsiveness without changing the actual workflow.
