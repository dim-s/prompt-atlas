# Wording anti-patterns and fixes

Scan a Claude-facing prompt for these common issues. Each has a concrete fix.

---

## 1. "CRITICAL / YOU MUST" overused

Older Claude prompts needed aggressive emphasis to fight undertriggering. On 4.5/4.6/4.7 this causes OVER-triggering — the model obeys the instruction too eagerly and applies it beyond its intended scope.

**Signs**: multiple ALL-CAPS imperatives, "NEVER"/"ALWAYS" stacked in the same list, every rule marked IMPORTANT.

**Fix**:
- Reserve ALL-CAPS for genuine safety invariants (secrets, data loss, destructive ops).
- Convert most "YOU MUST X" to "X when Y".
- Delete "If in doubt, use X" — newer models already handle the "when" judgment.

---

## 2. Vague subagent / skill description

The description is the delegation trigger. Vague descriptions never trigger the subagent/skill.

**Weak:** `description: Reviews files for issues.`
**Strong:** `description: Reviews PRs for security vulnerabilities — injection risks, auth/authz flaws, secrets in code, insecure deserialization. Use proactively after any code change that touches auth, data access, or external inputs.`

**Fix checklist**:
- Concrete verb + object
- At least one "use when..." or "use proactively" clause
- Slight pushiness when undertriggering is the concern
- Boundary condition if a close competitor exists

---

## 3. Kitchen-sink CLAUDE.md / AGENTS.md

Files that document every convention, every library, every past decision.

**Signs**: > 300 lines; bullets that state things like "use clean code"; file-by-file descriptions of the codebase.

**Fix**: for each line, apply the pruning test — "if I delete this, does the assistant start making mistakes?" If no, delete. Move domain-specific workflows to skills. Move deterministic requirements to hooks.

---

## 4. Negative instructions without a positive alternative

`NEVER do X` leaves Claude guessing what to do instead.

**Weak**: "Never use ellipses."
**Fix**: "Your response will be read aloud by TTS, so never use ellipses. End sentences with periods, questions with question marks, pauses with commas."

Rule: name the alternative AND explain the reason.

---

## 5. Rules without the WHY

`Use snake_case.` Claude will follow it but won't generalize to adjacent cases.

**Fix**: "Use snake_case for all Python identifiers — variables, functions, method names, and modules — to match PEP 8 and the rest of this codebase."

The WHY lets Claude handle edge cases you didn't think of.

---

## 6. "Think harder" / step-by-step prescriptions

Writing multi-step plans that Claude should follow internally, rather than instructions for what to produce.

**Weak**: "First, think about A. Then consider B. Then evaluate C. Then check D. Then..."

**Fix**: ask for the reasoning you actually want, or raise effort. Examples:
- "Verify your answer against these criteria before finishing: X, Y, Z."
- "First identify the hardest sub-problem, then solve it before the easier parts."
- "List the cases where your reasoning could be wrong, then address each."

Trust the model's own reasoning — don't prescribe the steps.

---

## 7. Over-forcing progress updates on Opus 4.7

Old prompt: "After every 3 tool calls, summarize progress."

Opus 4.7 already calibrates progress updates well in long agentic traces.

**Fix**: remove scaffolding. Re-evaluate. If updates are genuinely under-calibrated for your UX, describe the *shape* of the update (length, section format, trigger condition) with an example, not the frequency.

---

## 8. Guessing tool parameters

Prompting that encourages "if unsure, guess parameter values" leads to hallucinated tool args.

**Fix**:
> "Never use placeholders or guess missing parameters in tool calls. If required parameters are missing, ask the user or discover them via other tools."

---

## 9. One massive rule block

All rules mushed into one unstructured paragraph.

**Fix**: wrap each category in its own XML tag.

```
<code_style>
- Use ES modules
- Destructure imports when possible
</code_style>

<testing>
- Prefer running single tests, not the whole suite
- Skip mocks in integration tests
</testing>

<workflow>
- Typecheck after a series of changes
- Run the linter before committing
</workflow>
```

---

## 10. Style conventions Claude already follows

`Write clean code.` `Follow DRY.` `Use meaningful variable names.` `Be thoughtful.`

Claude already does these. They bloat the file and push real, project-specific rules out of attention.

**Fix**: delete. Include only conventions specific to your project or that contradict defaults.

---

## 11. Role is a novella

`You are Dr. Emily Chen, a 42-year-old Stanford-educated senior software architect with 15 years of experience at FAANG companies, specializing in...`

Biographical novella adds no behavioral signal. It consumes tokens and prompts unwanted roleplay.

**Fix**: strip biography. "You are a senior backend engineer specializing in distributed systems." does the work.

**Do NOT confuse with methodological anchors.** A role block that looks long but *names specific frames* — time horizons, analytical lenses, named frameworks ("2nd/3rd-order effects", "У-Вэй", "behavioral economics", "10–30 year horizon") — is earning its keep. Especially true for:
- Generative/advisory prompts (strategy, therapy-adjacent, creative writing), where research shows persona framing *improves* output (USC 2026, [arxiv 2603.18507](https://arxiv.org/abs/2603.18507))
- Literal models (Opus 4.7) that will NOT apply a "2nd-order effect" lens or a "10-year horizon" unprompted

The test: does the phrase name a **frame, horizon, method, or stance** the model can literally apply? Keep it. Is it just a **label of expertise** ("you are an expert X") or biography? Strip it. See `techniques.md` §6 for the full task-type breakdown.

---

## 12. Skill descriptions that undertrigger

`description: Helps with spreadsheets.`

Claude Code currently undertriggers skills — vague descriptions are especially prone to being skipped.

**Fix**:
- Name specific file extensions (`.xlsx`, `.csv`, `.tsv`).
- List concrete trigger phrases the user might use.
- Include "Use this skill any time..." or "Use this skill whenever..."
- Name the deliverable type.
- Add a boundary for close competitors.

---

## 13. SKILL.md body that duplicates every reference inline

If SKILL.md is 2000 lines, every trigger pays for all of it. Context is expensive.

**Fix**: move detail to `references/*.md`; reference them from SKILL.md as pointers. Move reusable logic to `scripts/*.py`. Keep SKILL.md under ~500 lines.

---

## 14. No "when NOT to use" for close competitors

If two skills have overlapping triggers, Claude may pick the wrong one.

**Fix**: add explicit exclusions in each description.
> "Do NOT trigger when the primary deliverable is a Word document (use docx skill), PDF (use pdf skill), or HTML report."

---

## 15. Matching prompt style to wrong output

Heavy-markdown prompt → heavy-markdown output. Dense bullet list prompt → dense bullet list output. The prompt style leaks.

**Fix**: write the prompt in the style you want the output to be in. Plain prose in → plain prose out.

---

## 16. Old frontend-design scaffolding on Opus 4.7

Earlier models (pre-4.7) needed long "avoid AI slop" prompt blocks. Opus 4.7 generates distinctive frontends with much less guidance.

**Fix**: if migrating, strip the long snippet and use the short version:
```
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

---

## 17. No verification / success criteria

Prompts that ask for work without defining what "done" looks like.

**Fix**: include the verification path — tests to run, expected outputs, screenshots to compare, a script to invoke, or a checklist to verify against. The single highest-leverage change for agentic coding prompts.

**Opus 5 caveat**: this stays true for *task* verification (what "done" means, which tests prove it). It does **not** license a self-review instruction — see anti-pattern 37.

---

## 18. Leaky abstractions across CLAUDE.md and subagents

Rules duplicated in CLAUDE.md and a subagent, or across multiple skills. They drift out of sync and eventually disagree.

**Fix**: one source of truth per rule. If CLAUDE.md has the rule, subagents just say "follow project conventions in CLAUDE.md". If a subagent owns the rule, CLAUDE.md doesn't repeat it.

---

## 19. Treating CLAUDE.md and AGENTS.md as separate sources of truth

Having the same content in both files, maintained separately, guarantees drift.

**Fix**: put the portable rules in AGENTS.md. In CLAUDE.md, `@AGENTS.md` at the top and then add ONLY Claude-specific overrides and workflows below.

---

## 20. Abstract / aspirational language

Lines like "be thoughtful", "be careful", "think holistically", "consider the user" do nothing — they're unmeasurable.

**Fix**: replace with specifics.
- "Be thoughtful" → "Before editing, list the functions called by this code"
- "Be careful with migrations" → "For any schema change, add a new migration file; never edit existing migrations"
- "Consider the user" → "Write error messages that tell the user what went wrong AND what to do next"

**Do NOT flag as this anti-pattern**: methodological anchors in a role/competency block that *look* abstract but name concrete frames the model can literally apply. Examples:

- ✅ "Systems synthesis (2nd/3rd-order effects)" — names the lens and its depth
- ✅ "10–30 year strategic horizon" — measurable time scope
- ✅ "Objective mirror without cognitive biases" — stance, pairs with Zero-Flattery rules elsewhere
- ✅ "Daoism as technology: У-Вэй, path of least resistance, asymmetric strategies" — named framework with specific primitives

These are load-bearing on literal models (Opus 4.7 in particular): the model won't apply a "10-year horizon" or a "2nd-order effects" lens unless it's named. Research supports this for generative/advisory tasks — see USC 2026 ([arxiv 2603.18507](https://arxiv.org/abs/2603.18507)) and `techniques.md` §6.

**The test** — for each line in a competency/role block, ask:
- Does it name a *frame, horizon, method, or stance* the model can apply? → Keep.
- Is it a vibe ("be thoughtful", "think holistically", "deep expertise")? → Drop.

Reviewer failure mode to avoid: seeing a bullet list of competencies, recognizing the *shape* of anti-pattern #20, and stripping the block without checking whether each item is a genuine anchor. This mistake costs the user a methodological frame they deliberately installed and then has to re-diagnose when the assistant stops applying it.

---

## 21. Front-loading warnings ahead of instructions

When the top of the prompt is a wall of "DON'T do this. DON'T do that.", Claude spends attention on exclusions before it knows what the task even is.

**Fix**: lead with the goal, the role, and the workflow. Put constraints and exclusions near the bottom, where recency bias helps them stick.

---

## 22. Conditional phrasing that hedges

"Maybe try to..." "If possible..." "It might be good to..."

Claude reads hedging as optional. Results are unpredictable.

**Fix**: be declarative. "Do X when Y." If the rule is genuinely conditional, state the condition clearly: "If the file is larger than 1000 lines, split it into modules before editing."

---

## 23. Asking yes/no questions that aren't yes/no

"Can you implement auth?" "Could you refactor this?"

Claude sometimes reads these as requests for an opinion rather than a task.

**Fix**: use imperatives. "Implement auth." "Refactor this." If you genuinely want opinion, say so: "Should we implement auth here? Explain trade-offs first, then wait for my decision."

---

## 24. No permission to say "I don't know"

Prompts that implicitly or explicitly demand an answer leave the model no graceful path when it's uncertain. It will hallucinate confidently rather than admit uncertainty.

**Signs**: no fallback path stated; phrases like "always answer", "never refuse to answer", "give your best guess"; demand for a specific output format with no "not applicable" branch.

**Fix**: add explicit permission to express uncertainty.

> "If you're not sure about something, say so clearly rather than guessing. 'I don't have enough information to answer this with confidence' is a valid response."

This is one of the most reliable anti-hallucination techniques per Anthropic's 2026 guidance.

---

## 25. Over-structured simple prompt

Wrapping a 3-sentence instruction in 5 nested XML tags, a role declaration, a constraints block, and an output-format section.

**Signs**: ratio of structural markup to content is high; the structure is elaborate but the instruction is short and self-contained.

**Fix**: delete the scaffolding. A single-paragraph imperative often outperforms the "heavily structured" version on modern Claude. Save XML structuring for prompts where there are genuinely multiple distinct parts (instructions + context + examples + long data).

---

## 26. The mega-prompt copy-paste

Starting from a 2000-word template found online, then tweaking one sentence for the current task.

**Signs**: lots of ALL-CAPS rules, multiple nested XML blocks, several role declarations, long lists of "NEVER do X" patterns, "take a deep breath", "you are an expert", "do your absolute best work".

Most of it is performative — none of it is actually tuned to the current task, but each rule eats attention that should be on the real requirements.

**Fix**: throw away the template. Start from a 3-line version — role + task + output format. Add complexity only when you see a specific failure. Anthropic's 2026 guidance is explicit: *"The best prompt isn't the longest or most complex. It's the one that achieves your goals reliably with the minimum necessary structure."*

---

## 27. "Take a deep breath" / "you are an expert" / motivational prefixes

Relics of earlier model generations where cheap psychological framing sometimes nudged behavior. On 4.5+ these are neutral-to-slightly-negative — they waste tokens, signal prompt laziness, and don't improve output.

**Fix**: delete. The same attention goes further spent on task-specific detail.

---

## 28. Persistent instructions buried in the middle

In a long CLAUDE.md or AGENTS.md, rules placed in the middle are more vulnerable to context-rot performance degradation than rules at the top or bottom (Paulsen 2025, Veseli et al. 2025). Claude attends to edges more reliably than middles, especially as context fills.

**Fix**: put genuine invariants (security, data loss, key conventions) at the TOP or BOTTOM of the file. Trim the middle aggressively; what survives there should be straightforward reference material, not load-bearing rules.

---

## 29. Process-step prescription for GPT-5.x

Lining up explicit steps the model must follow ("First do X, then Y, then Z…") on a GPT-5.x prompt — especially for 5.5 — actively degrades output. OpenAI's prompt-guide is explicit: *"GPT-5.5 treats detailed step-by-step instructions as interference: redundant instructions create noise, narrow the solution space, and make responses overly mechanical."*

**Signs**: numbered step lists in the system prompt; "First..., then..., then..." chains; methodology described as "the way to do this is to A → B → C → D".

**Fix**: replace with outcome + success criteria. Mention sub-steps as guidance only when the order is genuinely load-bearing.

| Weak (process-prescription) | Strong (outcome-first) |
|---|---|
| "First read the file. Then identify the bug. Then fix it. Then write a test." | "Fix the bug in `<file>` so the existing tests pass plus a new regression test you add." |
| "Step 1: validate. Step 2: query. Step 3: format." | "Endpoint: 200 on valid, 404 on missing, 400 on malformed." |

**When to keep step prescription**: order genuinely matters for safety (`techniques.md` §15) — "lock the file BEFORE reading", "backup BEFORE destructive op". The order *is* the rule; keep it.

**Cross-vendor case** (Claude + GPT-5.x): use the compromise from `techniques.md` §27 — outcome-first with steps as "typical sub-tasks, choose order yourself".

---

## 30. Porting old GPT prompts forward without a rewrite

Taking a GPT-5.4-era system prompt and running it on GPT-5.5 produces silent quality regressions. OpenAI's GPT-5.5 migration guide is explicit: *"treat it as a new model family to tune for, not a drop-in replacement… Begin migration with a fresh baseline instead of carrying over every instruction from an older prompt stack."*

The accumulated patches in an aged prompt — added to fix specific 5.4 quirks (overengineering, hallucinations on certain framings, format drift) — each cost attention on 5.5 and address problems the new model doesn't have. Net result: the old prompt is *worse* on the new model than a clean rewrite.

**Signs**: a long system prompt with comments like "added to fix X bug" or layers of "DO NOT do Y unless..." patches; the user reports "we just upgraded to 5.5 and behavior got weirder".

**Fix**: rewrite from a fresh baseline. Keep only the parts that address timeless concerns (safety invariants, domain-specific conventions). Drop quirks-specific patches; re-test on 5.5; add new patches only for failure modes you actually observe on 5.5.

**Caveat (Prime directive)**: when rewriting someone else's accumulated prompt, you can't always tell which patches were quirk-fixes and which were load-bearing for legitimate reasons. List them under Assumptions before deleting.

---

## 31. "Think step by step" instead of `reasoning_effort`

Pre-2025 prompting wisdom: prepend "Think step by step" to nudge the model into more careful reasoning. On 2026 models — both Claude 4.x and GPT-5.x — this is largely obsolete and sometimes counterproductive.

**Signs**: a "think step by step" / "let's think this through carefully" / "take a deep breath and reason through" line in the prompt; a multi-paragraph chain-of-thought scaffold inside a prompt that calls a reasoning-capable model.

**Fix**:
1. Check what `reasoning_effort` (GPT-5.x) or `effort` (Claude) the user is running. If `low`/`none`, raise it before changing the prompt — that's the actual lever.
2. If reasoning depth still isn't enough at high effort, escalate to a more capable model.
3. Replace prose with specific reasoning requests if you want them: "Verify against these criteria first: [...]", not "think harder".

See `techniques.md` §30 for the full pattern.

---

## 32. JSON shape described in prose instead of `json_schema`

When the task requires a strict output shape, describing the shape in the system prompt — fields, types, constraints, "do not include other fields", "do not wrap in markdown" — competes for attention and still allows drift. Both Anthropic and OpenAI APIs support structured outputs (`json_schema` with `strict: true`); push the format there.

**Signs**: a paragraph in the system prompt enumerating JSON fields and their constraints; trailing reminders like "Make sure age is between 0 and 150" or "Do not include other fields"; few-shot examples demonstrating the JSON shape.

**Fix**: move the format to `response_format: { type: "json_schema", json_schema: {...}, strict: true }`. The system prompt drops the format paragraphs and focuses on behavior. See `techniques.md` §28.

**When the user's stack doesn't support `json_schema`**: keep ONE tight prose contract; don't repeat the format in prompt + few-shot + reminder.

---

## 33. Tool-specific guidance crammed into the system prompt (GPT-5.x)

OpenAI's prompt guidance is explicit: *"Put most tool-specific guidance in the tool descriptions themselves: what the tool does, when to use it, required inputs, side effects, retry safety, and common error modes. Add tool-specific context to system instructions only when it applies across tools."*

**Signs**: long system prompt with sections like "When using the Slack tool, do X / When calling the database tool, do Y / The notification tool requires Z..."; per-tool usage hints accumulating in `AGENTS.md` rather than tool description fields.

**Fix**: move per-tool guidance into the tool's `description` schema. The system prompt then only carries cross-tool rules ("never call destructive tools without confirmation").

This applies to:
- OpenAI function-calling tool descriptions
- MCP server tool descriptions (both Claude Code and Codex CLI)
- Subagent tool definitions

**Why it matters more on GPT-5.x than Claude**: Claude tolerates per-tool guidance in the system prompt; GPT-5.x explicitly recommends against it. Cross-vendor `AGENTS.md` should follow the stricter (OpenAI) rule.

---

## 34. "Act as expert X" / persona-first framing on GPT-5.x

Persona prompts ("You are a senior X with N years of experience...") are weakening across all current models, but on GPT-5.x they're noticeably weaker than outcome-first phrasing. Community consensus on 5.5 in particular: outcome > role.

**Signs**: opening lines like "You are an expert at...", "Act as a..."; persona that describes credentials rather than naming methodological frames.

**Fix**: replace with outcome-first wording. "Return an analysis of the three biggest risks in this contract" beats "You are an experienced corporate lawyer; review this contract".

**Do NOT confuse with methodological anchors** (see anti-pattern #11 and `techniques.md` §6). A role block that names *specific frames* — "Apply 2nd/3rd-order effects analysis", "10–30 year horizon", "first-principles reframe" — is load-bearing, even on GPT-5.x. Strip credential-naming, keep frame-naming.

**The test**: does the line name a *frame, horizon, method, or stance* the model can apply? Keep. Is it just "expert at X" / years of experience / FAANG biography? Drop.

---

## 35. Codex AGENTS.md silently truncated by 32 KiB cap

Codex enforces `project_doc_max_bytes` (32 KiB by default) on the combined `AGENTS.md` hierarchy. When the hierarchy exceeds the cap, files past it are silently dropped — no warning, no error.

**Signs**: a deep-tree `AGENTS.md` rule that "doesn't seem to apply"; large repo with many sub-directory `AGENTS.md` files; a long global `~/.codex/AGENTS.md` plus a long repo-root `AGENTS.md`.

**Fix**:
1. Confirm the file is being read at all — check the combined hierarchy size.
2. Trim global AGENTS.md (personal preferences) so the budget is available for project rules.
3. Move reference-grade content (architectural notes, full convention docs) out of `AGENTS.md` and into linked docs the agent can read on demand.
4. Sub-directory `AGENTS.md` files should hold *deltas*, not duplicate inherited rules.

**Realistic budget**: target combined hierarchy under 8 KiB; reserve room for growth without surprise truncation.

---

## 36. Codex subagent body prescribing reasoning steps when `model_reasoning_effort` is high

A Codex subagent with `model_reasoning_effort: high` is already reasoning deeply by default. Body prose like "First identify the hardest sub-problem, then list constraints, then evaluate each candidate" duplicates what the model is already doing — and wastes attention on the *prescription* rather than the *outcome*.

**Signs**: subagent frontmatter has `model_reasoning_effort: high` (or `xhigh`); body has multi-step "how to think" prescriptions.

**Fix**: drop step prescriptions from the body. Keep success criteria, output format, and anti-failure-mode snippets. The reasoning depth is the parameter's job; the body specifies *what* to produce, not *how* to think.

**Inverse case**: a Codex subagent with `model_reasoning_effort: low` or `none` performing a task that genuinely needs reasoning — *that* one benefits from explicit step prompts in the body, because the parameter isn't doing the work. Wording fix in that case is "either raise effort or keep the step structure".

---

## 37. Carried-over self-verification instructions on Claude Opus 5

The Claude-family analog of #36: the model already does the thing the prompt is paying for. Opus 5 catches and fixes its own mistakes without being told to, so instructions like "double-check your answer", "re-verify before responding", "include a final verification step for any non-trivial task", or "use a subagent to verify" compound with its own behavior — producing over-verification that costs tokens and latency with no quality gain.

This one is unusually common because it was *correct advice* on every earlier Claude, so it's sitting in most prompts written before July 2026 (`techniques.md §20`).

**Signs**: target is Opus 5; the prompt or its harness contains a standalone self-review, double-check, or verifier-subagent step that isn't tied to a concrete artifact (a test suite, a schema, a tool result).

**Fix**: remove it rather than soften it — Anthropic's migration guidance is explicit about removal. Keep verification that belongs to the task ("run the test suite and report failures", "check each claim against a tool result from this session"); strip verification that's aimed at the model's own confidence.

**Do not generalize**: on Fable 5, Opus 4.8/4.7, Sonnet 5/4.6, and Haiku 4.5 the instruction still earns its keep. In a universal-Claude prompt, phrase verification as part of the task instead of as a self-review pass.

---

## 38. Telling the model not to think

Lines like "don't overthink this", "answer directly without reasoning", or "do not think before responding" fail in three distinct ways.

As a reasoning-depth control they're inert prose — the lever is the parameter (#31). Worse, on **Opus 5 with thinking disabled** such a rule measurably *increases* leakage of internal XML tags (`<thinking>` and friends) into the visible response. The instruction produces the opposite of its intent.

And on several models it **cannot be honored at all — the off-switch simply does not exist**: **Moonshot Kimi K3 / K2.7-Code** (K3 *"always has thinking enabled"* — depth via `reasoning effort`; K2.7-Code *"forces thinking and preserve_thinking as True"*), **Z.ai GLM-5.3** (`thinking.type` only accepts `enabled`; `disabled` requests fail), **xAI Grok 4.5 / 4.6** (reasoning cannot be disabled; the knob is `reasoning_effort`), and the **Qwen3.8 open-source weights** (`Qwen3.8-2.4T-A95B` requires thinking for all interactions). The prompt asks for a state the model cannot enter — worse than inert: a visible mismatch between the instruction and every response. Kimi started this pattern in the atlas; as of August 2026 the same constraint covers four vendor families, so strip the pattern unconditionally rather than per-target.

**Fix**: delete the rule. If the goal was cost or latency, lower `effort` out-of-band — on Opus 5, thinking enabled at `low` effort outperforms thinking disabled at comparable cost. If the goal was clean output from an integration that must keep thinking off, use the general form and **don't name the tags** (naming them is less effective than a blanket rule):

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

**Related artifact**: with thinking disabled, Opus 5 can also write a tool call into its text output instead of emitting a `tool_use` block. The call never runs, and in agentic loops the leaked text stays in history and affects later turns — most common on tool-heavy workloads like search. The permission-to-speak-first clause above is the documented mitigation.
