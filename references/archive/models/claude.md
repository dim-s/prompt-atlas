# Archive — legacy sections from `models/claude.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

## Claude Opus 4.7

The most important shifts versus older Claude generations. **Claude Opus 4.8 is covered by this section** — Anthropic's 4.8 guidance is a superset of 4.7's with no wording reversals; the 4.8-specific notes are called out inline. Both are now legacy-table models (current Opus tier is Opus 5, above).

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

Most advice from the Opus 4.7/4.8 section applies to Sonnet 4.6 too. The main differences:

- Sonnet 4.6 is a bit less literal than Opus 4.8/4.7 — it still generalizes a little more. Explicit scope is still helpful but less critical.
- Tone is similar to Opus 4.8/4.7 (direct, less validation-forward).

### When to use Sonnet 5 vs Sonnet 4.6 vs Opus 5

(A model-choice question that affects wording only indirectly.) **Sonnet 5** is a same-price capability upgrade over 4.6 and the default choice for new Sonnet work — it's an option for workloads needing more than 4.6 without moving to Opus-class. Keep **Sonnet 4.6** only where you specifically need sampling-parameter tuning (Sonnet 5 rejects it) or haven't re-tuned `max_tokens` for the new tokenizer yet. **Opus 5** is now Anthropic's default recommendation for complex agentic coding and wins for multi-hour autonomous work, deep research, large-scale migrations, and code review; it supersedes Opus 4.8 for new work. Between Sonnet 5 and 4.6, Sonnet 5 wins for interactive sessions, high-volume repetitive work, and anything speed-sensitive.

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
