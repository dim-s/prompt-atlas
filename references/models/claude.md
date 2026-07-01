# Model-specific wording differences

What actually changes about how you should PHRASE prompts for each current Claude model. This reference stays focused on wording — not API parameters, context sizes, or pricing.

---

## Claude Fable 5

New tier above Opus (June 2026; first public release of the Mythos line). Anthropic's guide is explicit that Fable 5 responds to the same prompting techniques as other Claude models — the deltas below are what actually changes for wording. Source: "Prompting Claude Fable 5" (official).

### A brief instruction replaces an enumeration

Instruction-following is strong enough that one short principle steers as well as listing every behavior by name. Prompts that grew long fighting older models' lapses now add noise. Example from the official guide — instead of enumerating each verbosity pattern to avoid:

> Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find". Supporting detail and reasoning come after. Being readable and being concise are different things, and readability matters more.

Same for checkpoint behavior — one boundary statement instead of case-by-case rules:

> Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide.

### Never instruct it to echo its reasoning

**The highest-severity Fable-specific finding.** Prompts, skills, or harness instructions that tell the model to echo, transcribe, or explain its internal reasoning as response text ("show your thinking", "explain your reasoning step by step in the answer") trigger the `reasoning_extraction` refusal classifier and cause fallbacks to Opus 4.8. When migrating, audit every skill and system prompt for reflection / show-your-thinking instructions. If reasoning visibility is needed, that's an API concern (summarized thinking blocks) — handoff, not a wording fix.

### Trim over-prescriptive skills and prompts

Skills developed for prior models are often too prescriptive for Fable 5 and **can degrade output quality**. Review and consider removing older instructions if default performance is better. This is the same "scar tissue" audit as the prime directive, with the direction reversed: on Fable 5, the burden of proof shifts toward removal.

### Subagent default flips: it delegates readily

Opposite of Opus 4.8/4.7. Fable 5 dispatches parallel subagents more readily and dependably sustains long-running ones. Wording shifts from encouragement to boundaries:

> Delegate independent subtasks to subagents and keep working while they run. Intervene if a subagent goes off track or is missing relevant context.

Prefer asynchronous orchestration over blocking on each subagent; for verification, fresh-context verifier subagents outperform self-critique.

### Long runs: ground claims, state boundaries, prevent overplanning

Three snippets cover the documented long-run failure modes.

Fabricated status reports (Anthropic: nearly eliminated them in testing):

> Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly.

Unrequested actions (drafting an email no one asked for, defensive git branches):

> When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one.

Overplanning on ambiguous tasks:

> When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue. If you are weighing a choice, give a recommendation, not an exhaustive survey.

### Rare early stopping and context-budget anxiety

Deep into long sessions Fable 5 can end a turn with a statement of intent ("I'll now run X") without the tool call, or offer to summarize and start a new session when the harness shows a remaining-token countdown. Wording fixes: for autonomous pipelines add an "operating autonomously — check your last paragraph before ending the turn" reminder; avoid surfacing context-budget counters to the model, or add "You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits."

### Give the reason, not only the request

Fable 5 performs better when it knows the intent behind a task — it connects the work to relevant context instead of inferring intent:

> I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].

### Memory pays off

Fable 5 is notably good at writing and using file-based memory. For recurring agents, provide a place for lessons and describe the hygiene:

> Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.

### Quiet between tool calls — narration flipped vs Opus 4.8

*Field observation (maintainer, Jun 2026), consistent with the official guide's framing.* Fable 5 narrates noticeably less between steps than Opus 4.8 — it silently reads, edits, and moves on. The official guide treats between-call text as optional ("terse shorthand is fine between tool calls… brevity there is good") and recommends a send-to-user tool for long runs precisely because little surfaces by default.

Two consequences for migrated prompts:

- **Strip 4.8-era silence-defaults.** Snippets like "Default to silence between tool calls; do not narrate routine actions" were added to quiet Opus 4.8's narration. On Fable 5 they double-suppress an already-quiet model.
- **If you want visibility in interactive sessions, ask for the shape, not a schedule:**

> Before your first tool call, say in one sentence what you're about to do. While working, give a brief update when you find something load-bearing or change direction — one sentence each. Routine reads and edits don't need narration.

For long asynchronous agents, narration is the wrong channel anyway — that's the send-to-user tool's job (harness concern, handoff).

### Final-summary readability in agentic sessions

In long tool-heavy sessions Fable 5 can carry working shorthand (arrow chains, invented labels, references to thinking the user never saw) into the user-facing summary. A communication addendum fixes it — key line: *"Terse shorthand is fine between tool calls; your final summary is for a reader who didn't see any of that. Drop the working shorthand, write complete sentences, open with the outcome."*

### Effort and safety domains — surface out-of-band, don't embed

- `effort` stays the primary lever (API/CLI parameter — never paste into the prompt body). Note for advice: `low` on Fable 5 often matches or beats `xhigh` on prior Opus — reflexive maximizing wastes latency and tokens.
- Prompts in offensive-cybersecurity or biology/life-sciences territory can trigger safety classifiers even on benign work — no wording fix exists; the answer is fallback configuration (API concern, handoff to `claude-api`).

---

## Claude Opus 4.7

The most important shifts versus older Claude generations.

### It's more literal

Opus 4.7 interprets instructions more literally than Opus 4.6 did. It will NOT silently generalize an instruction from one item to the next. It will NOT infer requests you didn't make.

**Practical wording fix**: state scope explicitly.

| Weak | Strong |
|---|---|
| "Format the section." | "Apply this formatting to every section, not just the first one." |
| "Fix the typo." | "Fix all typos in the file, not only the one mentioned." |
| "Add tests." | "Add tests for every public method in this file. Do not stop after the first few." |

This applies especially at low effort / when thinking is off. At higher effort the model is less strict about literalism but still won't make leaps the wording didn't sanction.

### Tone is more direct, less warm

Opus 4.7 is less validation-forward and uses fewer emoji than 4.6. If your product relies on a warm, collaborative voice, prompt for it explicitly:

> Use a warm, collaborative tone. Acknowledge the user's framing before answering.

If you don't care about tone, leave the default — it's concise and fine for most work.

### Response length calibrates to task

Opus 4.7 shortens responses for simple lookups and lengthens for open-ended analysis automatically. If you NEED a specific length regardless of task, say so — otherwise let it calibrate.

Verbosity-down wording when needed:

> Provide concise, focused responses. Skip non-essential context, and keep examples minimal.

Positive examples of desired concision work better than lists of things to avoid.

### It spawns fewer subagents by default

If you want aggressive subagent delegation, prompt for it:

> Do not spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see). Spawn multiple subagents in the same turn when fanning out across items or reading multiple files.

### It provides progress updates on its own in long agentic traces

Older prompts often included lines like "After every 3 tool calls, summarize progress". On Opus 4.7 this is redundant and sometimes counterproductive. Strip that scaffolding and see if progress updates look right unprompted. If not, describe the *shape* of update you want instead of the schedule.

### Code review prompts need "coverage" language

If you're writing a code-review prompt that says things like "only report high-severity issues" or "be conservative, don't nitpick", Opus 4.7 follows that more literally than older models did — meaning fewer reported findings.

Fix: tell the review step its job is coverage, not filtering.

> Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage — a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.

### Creativity, boldness, and "додумывание"

A common migration complaint on 4.7: "the model feels flatter / less creative / won't propose alternatives." This is a direct consequence of literalism — Anthropic's migration guide is explicit: *"if you've been counting on Claude to read between the lines, for example where you hint at tone and let it fill in the rest, the output will feel flatter."*

**First check: effort.** At `low` and `medium`, 4.7 "scopes its work to what was asked rather than going above and beyond" (migration guide). No prompt change overcomes low effort. Raise to `high` or `xhigh` before rewriting prompts.

**Then prompt-side fix: apply universal unlockers** — see [techniques.md §25](techniques.md). The five patterns (force-divergence "N options before a choice", permission to push back, divergent→convergent phases, steelman, first-principles reframe) are domain-agnostic and compose cleanly with project-specific methodological anchors (techniques.md §6).

**The combination matters**. Universal unlockers give the model *freedom of form*. Methodological anchors give it *direction*. Alone, each is incomplete on 4.7:
- Anchors without unlockers → focused but timid; converges on first-obvious answer.
- Unlockers without anchors → bold but unfocused; proposes creative-but-off-brief options.

For prompts where creativity matters (advisory, strategy, design, writing), prefer combining one unlocker + one or two methodological anchors over stacking multiple unlockers.

**When the WHOLE agent domain is creative** (game design, narrative, content, brand, coaching, brainstorming partner, design crit — see detection signals in techniques.md §25 "Whole-agent creative kernel"), per-task unlockers force the user to retype them every turn. Install the kernel once at the system-prompt level instead. Same combination logic, different placement: kernel sits in the agent's permanent role text, anchors stay project-specific.

Detection cues — recognize a creative agent prompt even without an explicit "creative" label:
- Primary output is ideas / options / critique / narrative / pitch material
- Domain keywords: game design, narrative, screenwriting, content, brand, strategy, coaching, brainstorming, advisory, design
- User symptoms: "feels mechanical", "won't push back", "лишь буллетами отвечает", "сухо", "не додумывает", "без огонька", "потерял тёплость", "выдаёт один очевидный ответ"

Effort-level nuance specifically for creative agents — `xhigh` is NOT a universal win because it makes the model self-edit harder and prunes the weird options brainstorming wants. Use `high` as the creative default; `medium` for raw divergent brainstorm; `xhigh` only when the creative task has a structured frame (memo, GDD section, plotline against constraints). Full table in techniques.md §25.

### Design default (frontend work)

Opus 4.7 has a strong default visual style: warm cream backgrounds, serif display type, terracotta accents. Great for editorial, hospitality, and portfolio briefs — wrong for dashboards, dev tools, fintech, healthcare.

Two effective fixes:

1. **Specify a concrete alternative palette and typography.** Opus 4.7 follows concrete specs precisely.
2. **Ask the model to propose 4 distinct directions first**, then pick one:

> Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface — one-line rationale). Ask the user to pick one, then implement only that direction.

### Old frontend-design scaffolding can be trimmed

Earlier models needed long "avoid AI slop" prompt blocks. Opus 4.7 does better with a much shorter snippet:

```
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

---

## Claude Sonnet 5

Current Sonnet-tier frontier (June 2026; `claude-sonnet-5`). A **drop-in upgrade for Sonnet 4.6** — Anthropic's guide is explicit it "performs well out of the box on existing Claude Sonnet 4.6 prompts." What actually changes for wording is a shift toward Opus-family literalism plus three API-level changes. 1M context (default and max, no smaller variant), 128k max output, priced the same as 4.6 ($3/$15; intro $2/$10 through Aug 31 2026). Sources: "Prompting Claude Sonnet 5" + "What's new in Claude Sonnet 5" (official).

### The behavioral headline: Sonnet 5 moved toward Opus literalism

The old mental model — "Sonnet 4.6 is less literal than Opus, generalizes a bit more" — no longer describes Sonnet 5. It "interprets prompts literally and explicitly, particularly at lower effort levels. It does not silently generalize an instruction from one item to another, and it does not infer requests you didn't make." Same wording fix as Opus 4.7/4.8: state scope explicitly.

| Weak | Strong |
|---|---|
| "Format the section." | "Apply this formatting to every section, not just the first one." |

The upside is precision — it performs better for API use cases with carefully tuned prompts, structured extraction, and predictable pipelines.

### Adaptive thinking on by default (change from 4.6)

On Sonnet 4.6, a request with no `thinking` field ran WITHOUT thinking. On Sonnet 5 the same request runs WITH adaptive thinking (turn off via `thinking: {type: "disabled"}` — an API concern). Manual extended thinking (`budget_tokens`) is removed and returns 400, same as Opus 4.8/4.7.

Wording implication: adaptive-thinking triggering is **steerable from the prompt**. If the model emits thinking blocks more often than you want — common under large or complex system prompts — add:

> Thinking adds latency and should only be used when it will meaningfully improve answer quality, typically for problems that require multi-step reasoning. When in doubt, respond directly.

### Effort: default `high`, respects levels strictly

Effort defaults to `high` (same as 4.6); raise to `xhigh` for the hardest coding/agentic tasks. Sonnet 5 "respects effort levels strictly, especially at the low end" — at `low`/`medium` it scopes work to exactly what was asked. If reasoning is shallow on a complex problem, **raise effort rather than prompting around it**. If you must hold effort at `low` for latency, add targeted guidance:

> This task involves multi-step reasoning. Think carefully through the problem before responding.

Rough cross-model mapping (migration, not wording): Sonnet 5 @ medium ≈ Sonnet 4.6 @ high; Sonnet 5 @ high ≈ Sonnet 4.6 @ max. Benchmark by observed thinking length, not effort name.

### More agentic — reaches for tools and self-verifies more readily

Sonnet 5 is "more agentic than Claude Sonnet 4.6 by default and will reach for tools and run self-verification loops more readily." Two wording consequences:

- **With thinking disabled** it's *less* likely to reach for tools — if you rely on tool calls with thinking off, add an explicit nudge in the system prompt.
- Effort is a tool-usage lever: `high`/`xhigh` show substantially more tool use in agentic search and coding.

Note: the guide documents readier tool use and self-verification; it does **not** document a subagent-spawning default flip like Fable 5's. Don't assume aggressive subagent delegation — matrix Table C marks Sonnet 5's subagent default conservatively.

### Verbosity calibrates to task

Like Opus 4.7, Sonnet 5 "calibrates response length to the complexity of the task rather than defaulting to a fixed verbosity" — shorter on simple lookups, longer on open-ended analysis. Same verbosity-down snippet, and positive examples beat negative lists:

> Provide concise, focused responses. Skip non-essential context, and keep examples minimal.

### Progress updates: strip the scaffolding

Sonnet 5 "provides regular, higher-quality updates to the user throughout long agentic traces." If you added "After every 3 tool calls, summarize progress" scaffolding, remove it. If the calibration is off for your use case, describe the update *shape* you want plus examples — same pattern as Opus 4.7.

### Sampling parameters removed — `temperature` / `top_p` / `top_k` → 400

**The biggest cross-vendor delta.** Setting `temperature`, `top_p`, or `top_k` to a non-default value returns a 400 error on Sonnet 5. The guide: *"This constraint is new for Sonnet-class models; the same constraint was previously introduced on Claude Opus 4.7."* The Claude models that now reject sampling tuning are **Fable 5, Opus 4.8, Opus 4.7, and Sonnet 5** — Sonnet 4.6 and Haiku 4.5 still accept it. Use system-prompt instructions for tone/variety instead of temperature. This moves the newest Claude models into the same camp as Gemini 3 (temperature fixed at 1.0) — see the cross-model note below.

### Design/frontend: propose-directions is now the variety lever

Sonnet 5 "may settle into a consistent default visual style" on open-ended frontend/design briefs. Generic negatives ("don't use that color") just shift it to another fixed palette. Because temperature is no longer accepted, **proposing options before building is the recommended way to get meaningfully different directions across runs**:

> Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.

The same `<frontend_aesthetics>` anti-slop block (see Opus 4.7 section) applies verbatim.

### Code review harnesses: coverage language (same as Opus 4.7)

A harness tuned for an older model may show lower recall on Sonnet 5 — a harness effect, not a capability regression. "only report high-severity issues" / "be conservative" is now followed more faithfully: the model investigates just as deeply but converts fewer investigations into reported findings (precision rises, recall can fall). Same coverage snippet as Opus 4.7:

> Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.

### New tokenizer — a token-budget concern, not a wording one

Sonnet 5 uses a new tokenizer: the same text produces **~30% more tokens** than 4.6. Not an API change (shapes unchanged), but `max_tokens` limits tuned for 4.6 may truncate equivalent output, token counts rise, and per-request cost shifts even though per-token price is unchanged. Surface as a migration note. At `high`/`xhigh`/`max`, leave `max_tokens` headroom for thinking + tool calls, or you get a response that's almost all thinking then a truncated answer with `stop_reason: "max_tokens"`.

### Tone/style may shift

Prose style on long-form writing may drift from 4.6; re-evaluate voice prompts against the new baseline. Warm-voice snippet:

> Use a warm, collaborative tone. Acknowledge the user's framing before answering.

### First Sonnet with real-time cybersecurity safeguards

Benign-but-adjacent security prompts may hit a refusal, returned as HTTP 200 with `stop_reason: "refusal"` (not an error). No wording fix — fallback handling is an API concern (handoff to `claude-api`).

---

## Claude Sonnet 4.6

The **previous** Sonnet — the current Sonnet frontier is Sonnet 5 (above). Still covered because it's widely deployed and, unlike Sonnet 5, accepts sampling-parameter tuning. Tone, literalism, and subagent-spawning defaults sit between Opus 4.7 and older Sonnet 4.5.

### It explores a lot by default

Sonnet 4.6 invests heavily in upfront exploration, especially on hard tasks. Prompts that used to say "be thorough" or "if in doubt, use [tool]" now cause OVER-exploration.

Fix: replace blanket defaults with targeted guidance.

| Was useful for old models | Now often over-triggers |
|---|---|
| "Always use X when in doubt." | "Use X when it would enhance your understanding of the problem." |
| "Be thorough; read all relevant files." | "Read the specific files you need; avoid exhaustive scans." |

### It tends to overengineer

Sonnet 4.6 will create extra files, add abstractions, and build in flexibility that wasn't requested. Counter with an explicit guard:

```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:
- Scope: don't add features, refactor, or make "improvements" beyond what was asked.
- Documentation: don't add docstrings, comments, or type annotations to code you didn't change.
- Defensive coding: don't add error handling or validation for scenarios that can't happen.
- Abstractions: don't create helpers for one-time operations or design for hypothetical future needs.
```

### Prompting language is similar to Opus 4.8/4.7

Most advice from the Opus 4.8 section applies to Sonnet 4.6 too. The main differences:

- Sonnet 4.6 is a bit less literal than Opus 4.8/4.7 — it still generalizes a little more. Explicit scope is still helpful but less critical.
- Tone is similar to Opus 4.8/4.7 (direct, less validation-forward).

### When to use Sonnet 5 vs Sonnet 4.6 vs Opus 4.8

(A model-choice question that affects wording only indirectly.) **Sonnet 5** is a same-price capability upgrade over 4.6 and the default choice for new Sonnet work — it's an option for workloads needing more than 4.6 without moving to Opus-class. Keep **Sonnet 4.6** only where you specifically need sampling-parameter tuning (Sonnet 5 rejects it) or haven't re-tuned `max_tokens` for the new tokenizer yet. **Opus 4.8** still wins for multi-hour autonomous work, deep research, and large-scale migrations. Between Sonnet 5 and 4.6, Sonnet 5 wins for interactive sessions, high-volume repetitive work, and anything speed-sensitive.

---

## Claude Haiku 4.5

The fastest and cheapest current model. Near-frontier intelligence at a small fraction of Opus cost.

### It's less forgiving of vague wording

Haiku 4.5 handles ambiguity less gracefully than Opus 4.8 / Sonnet 4.6. Prompts need to be MORE specific, not less.

- Spell out exactly what you want.
- Include examples when format matters.
- State the output shape explicitly.

### It's less spontaneous about tools

Opus and Sonnet tend to pick up tool usage well from context. Haiku is more conservative — if you want the model to use a tool, prompt for it explicitly.

| Weak (Haiku may skip the tool) | Strong |
|---|---|
| "Figure out what's in this file." | "Use the Read tool to open this file, then summarize it." |
| "Check the docs." | "Use WebFetch to load the URL above, then answer from that content." |

### It's well-suited to focused, bounded tasks

Haiku shines at: classification, extraction, formatting, quick read-only exploration, summarization, initial-pass review. Keep its scope tight. If you find yourself writing a 10-step workflow for a Haiku prompt, you're probably using the wrong model — promote to Sonnet.

### As a subagent

Haiku is the right default when the subagent's job is bounded and read-only: searchers, classifiers, extractors, cheap pre-filters. Use Sonnet or Opus for the main conversation where reasoning quality matters.

---

## Universal / multi-model prompts

Most CLAUDE.md and AGENTS.md files — and most skills and subagents used across different sessions — need to work across multiple models. The user of the project may be running Fable 5, Opus 4.8 or 4.7, Sonnet 5 or 4.6, or Haiku 4.5 on any given day, or even configure subagents to use different models. A universal prompt is not the same as a model-specific prompt with the model name stripped out.

### How to ask the user about model target

When it's unclear, ask one short question:

> "Под какую модель этот промпт — Fable 5, Opus 4.8 / 4.7, Sonnet 5 / 4.6, Haiku 4.5 или универсальный (должен работать на всех сразу)?"

If they don't know, the safest default is **universal** — write for the lowest-common-denominator behavior across 4.5+ models.

### Rules for writing universal prompts

**1. Write for the least forgiving model in the set.**

If Haiku 4.5 is in scope, be MORE specific — Haiku handles vague prompts worse than Opus. Spell things out concretely; add examples where format matters; name tools explicitly when you expect them to be used.

**2. Avoid leaning on single-model features.**

- Don't assume Haiku will do deep multi-step reasoning — break complex tasks into concrete steps in the prompt itself.
- Don't assume Opus's literalism to save you from having to state scope (Sonnet generalizes a bit more freely).
- Don't write tone instructions assuming one model's default ("be less formal" reads differently on Opus 4.8 vs Haiku).

**3. Be moderate with emphasis.**

"CRITICAL:" and "YOU MUST" are overread on 4.5+ and underread on older models. Avoid them as much as possible. When you need emphasis, use it sparingly and only on genuine invariants — that works acceptably across the whole range.

**4. Use structure rather than intensity.**

Instead of fighting for attention with ALL-CAPS, use XML tags or clear section headings. Structure is model-invariant; emotional intensity is not.

```xml
<invariants>
- Never commit .env or secrets.* files
- Always run the linter before committing
</invariants>

<style>
- Use ES modules, not CommonJS
- snake_case for Python
</style>
```

**5. Prefer positive framing and WHY-based rules.**

These work consistently across all models. Negative-only rules without alternatives and unexplained mandates are fragile — they work on some models and fail on others.

**6. Watch for model-specific scaffolding that should be stripped.**

- Old "avoid AI slop" long frontend prompts → trim for 4.7 / 4.8 (4.8 needs even less).
- "After every 3 tool calls, summarize" → Opus 4.7 already does this; still harmless on Sonnet 4.6 but adds noise.
- "Be thorough, use X tool when in doubt" → now causes overtriggering on 4.5+. Soften to "Use X when it enhances your understanding".
- "Show / explain your reasoning in the answer" → if Fable 5 may be in scope, strip: triggers the `reasoning_extraction` refusal classifier (see the Fable 5 section). Harmless-but-unnecessary on Opus/Sonnet/Haiku.

**7. Don't name a specific model in the prompt unless you have to.**

Naming Claude in a prompt that might run on any model locks you to that model. Instead of "You are Claude Opus 4.8", use "You are a helpful coding assistant". Reserve the model name for cases where identity matters (e.g. the user will see the model string).

**8. When a universal prompt isn't enough, split.**

If a prompt really needs different behavior per model, don't try to cram all variants into one. Instead:
- Write the baseline for the weakest model (usually Haiku).
- Add an "additional guidance" section at the bottom that cites specific model behaviors without mandating them.

Example:

> Additional: if you have adaptive thinking or extended reasoning available, use it to verify your answer against the success criteria before responding. If not, quickly re-read the question and your answer one more time before finalizing.

This lets stronger models take advantage of their features without breaking weaker ones.

### Universal prompt checklist

Before calling a prompt "universal", verify:

- [ ] No hard dependency on model-specific features (adaptive thinking, high-res vision, `xhigh` effort, etc.)
- [ ] Specificity high enough for Haiku 4.5 to follow
- [ ] Explicit scope stated for Opus 4.8/4.7 and Sonnet 5 literalism (Sonnet 5 is now Opus-literal, not 4.6-loose)
- [ ] Overengineering/over-exploration guards for Sonnet 4.6
- [ ] Emphasis used sparingly (no "CRITICAL:" stack)
- [ ] No model name hardcoded unless required
- [ ] Uses XML/headings for structure, not emphasis, to signal important parts
- [ ] If thinking-related: phrased as "consider" / "verify" / "reason through" rather than "think harder"
- [ ] No "echo / show / explain your reasoning" instructions (refusal trigger if Fable 5 is in scope)
- [ ] No `temperature` / `top_p` / `top_k` tuning assumed if Opus 4.7+/4.8, Fable 5, or Sonnet 5 is in scope (non-default returns 400) — steer tone/variety with wording instead

---

## Cross-model wording principles

These apply regardless of which model you're targeting:

### Sampling parameters are now fixed on the newest Claude models

`temperature`, `top_p`, and `top_k` at non-default values return a 400 error on **Fable 5, Opus 4.8, Opus 4.7, and Sonnet 5** (the constraint began on Opus 4.7 and reached Sonnet-class with Sonnet 5). Sonnet 4.6 and Haiku 4.5 still accept them. Practical consequences for wording:

- Don't design a prompt that relies on lowering temperature for determinism or raising it for variety on these models — the lever is gone. Steer tone and variety through the system prompt instead.
- For design/creative variety across runs, the "propose N options, then implement the chosen one" pattern replaces temperature (see the Sonnet 5 and Opus 4.7 design sections).
- This aligns the newest Claude models with Gemini 3 (temperature fixed at 1.0). In cross-vendor `AGENTS.md`, "don't reference temperature in the body" is now the safe rule for Claude *and* Gemini — only Sonnet 4.6 / Haiku 4.5 / GPT-5.x still tune it.

### Dial back aggressive language on all 4.5+ models

Prompts from the 4.0/4.1 era often stacked "CRITICAL:", "YOU MUST", "IMPORTANT:", "NEVER" to fight undertriggering. On 4.5+ this causes OVER-triggering — the model now obeys too aggressively and applies the rule beyond its intended scope.

- Keep all-caps only for genuine safety invariants (secrets, data loss, destructive ops).
- Convert most "YOU MUST X" to "X when Y".
- Drop "If in doubt, use X" — newer models handle the "when" judgment themselves.

### Prefer "consider" / "evaluate" / "reason through" over "think"

On some models (notably Opus 4.5 when thinking is disabled), the word "think" and variants can cause unexpected behavior. When you want careful reasoning but thinking is off, prefer:

- "Consider the edge cases"
- "Evaluate the trade-offs"
- "Reason through the options"

Not a big issue on 4.6+ but still a safer default when targeting multiple generations.

### Don't prompt for "think harder"

If reasoning is shallow, the wording isn't going to fix it — you need a more capable model (or higher effort setting, which is an API concern handled elsewhere). The wording fix is to ask for specific kinds of reasoning:

- "Verify your answer against these criteria before finishing: ..."
- "Before concluding, list the cases where your reasoning could be wrong."
- "First identify the hardest sub-problem, then solve it before the easier parts."

### Progress updates: describe the shape, not the frequency

Old prompts said "summarize every N tool calls". Newer models calibrate this themselves. If you really need a specific update shape:

> After each major phase (exploration, planning, implementation, verification), emit a 2-3 sentence status update in a `<progress>` block. Do not emit updates for minor tool calls.

This tells the model WHAT to produce, not WHEN — and the model handles the timing well.
