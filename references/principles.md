# Universal prompt-writing principles

Single source of truth for principles that apply across ALL vendors, models, and agentic systems. Vendor-specific deltas live in `models/<vendor>.md`. System-specific deltas live in `agentic-systems/<system>.md`. The differences live in `matrix.md`.

If a principle is genuinely universal, it lives here. If a principle is true for only some vendors/models/systems, it lives in the appropriate delta file.

When writing a review, read this file once, then drill into the relevant deltas.

---

## The 20 core principles

### Principle 1 — The shortest prompt that reliably gets the job done

Anthropic's 2026 guidance: *"the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome."* OpenAI's GPT-5.5 guidance is even stricter: *"redundant instructions create noise, narrow the solution space, and make responses overly mechanical."*

The load-bearing word is **reliably**. Don't trim phrasing whose effect on reliability you can't explain — see Prime directive in `SKILL.md`.

### Principle 2 — Be specific and direct

If a colleague without your context would be confused, the model will be too. Reference specific files, mention constraints, point to example patterns.

| Weak | Strong |
|---|---|
| "Add tests for foo" | "Write a test for `foo.py` covering the edge case where the user is logged out. Avoid mocks." |
| "Fix the login bug" | "Users report login fails after session timeout. Check `src/auth/`, especially token refresh. Write a failing test that reproduces, then fix." |

### Principle 3 — Explain the WHY, not just the rule

"Never use ellipses because TTS can't pronounce them" beats "NEVER use ellipses". The model generalizes from the reason.

### Principle 4 — Tell the model what to DO, not what NOT to do

"Respond directly, without preamble" beats "Don't start with 'Here is...'". Negative-only walls leave the model guessing what TO produce.

### Principle 5 — Match prompt style to desired output style

Markdown-heavy prompt → markdown-heavy output. Plain prose in → plain prose out. Style is contagious.

### Principle 6 — Use XML tags or clear section headings for multi-part prompts

Both Anthropic and OpenAI explicitly recommend XML for input structuring. OpenAI: *"using structured XML specs like `<[instruction]_spec>` improved instruction adherence."*

Less critical for short simple prompts where they just add overhead. For prompts mixing instructions + context + examples + data + constraints, structure is non-negotiable.

### Principle 7 — Examples are not free

For Claude, 3-5 diverse `<example>` tags remain a default tool for steering format/tone. For GPT-5.x reasoning models, OpenAI explicitly warns examples can hurt: *"few-shot prompts can reduce performance when the task requires heavy reasoning."*

Default decision tree: zero-shot first, add examples only when format is genuinely strict and zero-shot drifts. See `matrix.md` for vendor-specific tolerance.

### Principle 8 — Functional role, not biography

"You are a senior backend engineer specializing in distributed systems" does the work; "Dr. Smith, 42, Stanford, 15 years at FAANG..." is decoration.

GPT-5.x community consensus is stricter: outcome-first often beats persona ("return an analysis of X" outperforms "you are an expert at X"). See `models/gpt.md` and `matrix.md`.

### Principle 9 — Long documents at the top, the question at the bottom

Up to ~30% better long-context results. Compose with prompt-caching: stable content first, dynamic / user-specific tail — the cache invalidates on the first changed token.

### Principle 10 — Reserve aggressive emphasis for genuine invariants

"CRITICAL: YOU MUST" causes overtriggering on Claude 4.5+ (model applies the rule beyond intended scope) and is mostly inert noise on GPT-5.x. Save all-caps for safety invariants (secrets, data loss, destructive ops).

Anthropic's official Claude Code best-practices doc still says: *"You can tune instructions by adding emphasis (e.g., 'IMPORTANT' or 'YOU MUST') to improve adherence."* This is correct but easily abused. The reconciliation: emphasis works on **rare, load-bearing** rules; it stops working when every rule has it.

### Principle 11 — Permission to express uncertainty

"If you're not sure, say so rather than guessing" reduces hallucinations measurably on both vendors. For long-context Q&A, add: "If the answer isn't in the provided documents, say 'I don't know' rather than inferring."

### Principle 12 — Don't prompt for "think harder"

The phrase doesn't work on either vendor. The lever is the API parameter (`effort` on Claude, `reasoning_effort` on GPT-5.x). Wording substitutes:

- "Verify your answer against these criteria before finishing: [...]"
- "Before concluding, list the cases where your reasoning could be wrong."
- "First identify the hardest sub-problem, then solve it before the easier parts."

### Principle 13 — State scope explicitly

Modern reasoning models, especially Opus 4.7 and GPT-5.5, interpret instructions literally and **will not** silently generalize. If the rule applies to "every section, not just the first", say so.

### Principle 14 — Treat a new model version as a fresh baseline when the vendor says so

OpenAI is explicit for GPT-5.5: *"begin migration with a fresh baseline instead of carrying over every instruction from an older prompt stack."* Anthropic is more forgiving across Claude versions (forward-compatible).

When porting prompts: ask whether the vendor recommends fresh-baseline migration. If yes, the patches accumulated against earlier versions' quirks now address problems the new model doesn't have. Strip them.

### Principle 15 — Tool-specific guidance lives in tool descriptions, not the system prompt

OpenAI's strong split: *"Put most tool-specific guidance in the tool descriptions themselves... Add tool-specific context to system instructions only when it applies across tools."* Anthropic tolerates either; for cross-vendor portability, follow OpenAI's stricter rule.

### Principle 16 — Push output contracts into API parameters, not prompt prose

If output must be JSON of a specific shape, use `json_schema` with `strict: true` (GPT-5.x) or response format constraints (Claude). Frees the system prompt to focus on behavior, not formatting.

### Principle 17 — Persistent-context files (CLAUDE.md / AGENTS.md) should be short and specific

Context-rot research (Paulsen 2025, Veseli 2025) shows performance degrades as context fills, with the middle hit hardest. Every line in a persistent-context file earns its place.

Codex enforces this with a hard cap (`project_doc_max_bytes` defaults to 32 KiB; files past the cap are silently dropped). Claude Code has no equivalent hard cap but suffers context rot past ~300 lines. Anthropic's official guidance: *"Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"*

### Principle 18 — A skill / subagent description IS the delegation trigger

If it's vague, neither Claude nor Codex will delegate. Include "use when..." / "use proactively" clauses. Be slightly pushy if undertriggering. Add scope boundaries against sibling agents.

### Principle 19 — Start simple, add complexity only after observing a failure mode

Don't pre-add scaffolding for problems you haven't seen. Three similar lines is better than a premature abstraction.

### Principle 20 — Respect existing complexity (Prime directive — see SKILL.md)

When reviewing someone else's prompt, treat unusual phrasing, repetition, or strong negative instructions as evidence of past failure modes — not first-draft cruft. Default to the smallest change that fixes the finding. List unclear lines under "Assumptions / questions" rather than removing them silently.

---

## Verification — single highest-leverage technique

Anthropic's official Claude Code docs: *"Give Claude a way to verify its work — this is the single highest-leverage thing you can do."*

For any prompt with side effects (writes code, sends messages, mutates state), include verification:

- **Tests**: example test cases the output must satisfy
- **Visual diff**: screenshots compared before/after
- **Build / lint / typecheck**: a deterministic command that exits 0 on success
- **Schema validation**: a `json_schema` the output must match
- **Output contract**: the parent's downstream parser

When reviewing prompts that lack a verification path: this is almost always a `[CRITICAL]` finding for slash-commands and subagents, `[IMPROVE]` for ad-hoc prompts.

---

## Pattern: hint + literal anchor (preserving extrapolation on literal models)

Some prompts are intentionally hints-and-vibes style — named lenses ("apply KISS"), philosophical framing, atmospheric description. On older Claude (pre-4.7) the model extrapolated from vibes automatically; on Opus 4.7 and GPT-5.5 it doesn't.

The anti-pattern is to react by stripping hints and replacing with flat literal rules. That kills the frame the user was relying on.

The correct move is to **keep the hint AND add a literal anchor** — preserving extrapolation *and* giving the literal model a foothold:

- **Trigger clause**: *when* to invoke + *what observable output*. "Перед значимой правкой кода прогоняй через эти линзы; если линза подсвечивает — озвучь одной фразой до кода."
- **One concrete example** showing the hint applied to a real decision.
- **Permission to surface taste-uncertainty**: *"Если не уверен, какой из вариантов лучше — предложи списком, жди выбора. Не выбирай молча."*
- **Place philosophy at file edges** (top or bottom), not middle — edges survive context rot better.

When this pattern applies: target is a literal model (Opus 4.7 or GPT-5.5) AND the prompt contains deliberate hints-and-vibes framing. If the prompt is already all-literal-rules, skip — nothing to anchor.

Non-prompt lever: on Claude Code with Opus 4.7, raising `effort` to `xhigh` often restores hint-sensitivity better than any prompt edit.

---

## The CLAUDE.md ↔ AGENTS.md pattern

When both files exist:

```markdown
# Project instructions for Claude

@AGENTS.md

## Claude-specific

- Prefer Claude Code's built-in Explore subagent for codebase research before any larger refactor
- When using Plan Mode, save the plan to plans/ before switching to Normal Mode
- [Claude-specific tool/workflow notes]
```

AGENTS.md holds portable, tool-agnostic rules. CLAUDE.md imports it and adds Claude-specific overrides. Don't duplicate rules across both — they will drift. Pick one source of truth per rule.

Full AGENTS.md cross-tool semantics in `agentic-systems/_common.md`. Claude-Code-specific behaviors in `agentic-systems/claude-code.md`.

---

## Universal-prompt rules (cross-model, cross-vendor)

A prompt that must work across multiple models or vendors:

1. **Write for the most literal version in scope.** If Opus 4.7 or GPT-5.5 are in the set, default to outcome-first phrasing and explicit scope.
2. **Avoid leaning on single-model features.** No 1M-context behavior assumptions, no specific reasoning-knob defaults, no tone defaults that read differently across models.
3. **Use moderate emphasis.** All-caps and "CRITICAL:" only on genuine invariants — works acceptably across the whole range.
4. **Use structure rather than intensity.** XML tags or clear headings — model-invariant. Emotional intensity — not.
5. **Prefer positive framing and WHY-based rules.** Negative-only rules without alternatives are fragile.
6. **Don't hardcode model names.** "You are Claude Opus 4.7" or "You are GPT-5.5" locks you to one model. Use functional roles.
7. **When a universal prompt isn't enough, split.** Baseline for the weakest model + an "additional guidance" section that cites stronger-model behaviors without mandating them.

Detailed cross-vendor compromises: `models/_universal.md`. The differences themselves: `matrix.md`.

---

## Note on language

If the prompt is written in Russian (or another language) and addresses topic content in that language, the prompt text can stay in that language — the principles apply the same. But inside SKILL.md / subagent descriptions, **include English keywords too**, because the triggering heuristics on both Claude Code and Codex match English trigger phrases more reliably.

---

## When principles conflict

Two principles can pull opposite directions:
- "Shortest prompt" (#1) vs "preserve hidden intent" (#20)
- "Tell what to DO" (#4) vs "explicit scope / negative boundary" (#13, #18)
- "Examples beat paragraphs" (#7) vs "few-shot can hurt reasoning" (#7 GPT side)

Resolution rules:
- **Reliability wins over brevity**: a slightly-too-long prompt still works; a prompt missing its load-bearing clause doesn't.
- **Coverage wins over filtering at review time**: surface findings even when uncertain; the user filters.
- **Specificity wins for literal models**: when in doubt about Opus 4.7 / GPT-5.5, add scope rather than trust generalization.
