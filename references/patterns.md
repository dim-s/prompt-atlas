# Review patterns — hint + anchor, creative-domain kernel, small-model task prompt

Three install-once patterns that SKILL.md routes to. Each has detection cues, what to install, and when NOT to apply. Snippet text lives in `techniques.md` (§25 for the kernel) and `techniques-small.md` (skeleton) — copy from there, do not paraphrase.

---

## Pattern: hint + literal anchor

Some prompts are intentionally hints-and-vibes style — named lenses ("apply KISS / reuse-first"), philosophical framing, atmospheric description. On older Claude (pre-4.7) the model extrapolated from vibes automatically; from Opus 4.7 on — every current Claude — and on GPT-5.5 onward it doesn't.

The anti-pattern is to react by stripping hints. That kills the frame the user was relying on for generalization, tone, and taste. The correct move is to **keep the hint AND add a literal anchor** — preserving extrapolation *and* giving the literal model a foothold.

Four anchors to pair with load-bearing hints:

- **Trigger clause** — *when* to invoke + *what observable output*. "Перед значимой правкой кода прогоняй через эти линзы; если линза подсвечивает — озвучь одной фразой до кода."
- **One concrete example** showing the hint applied to a real decision.
- **Permission to surface taste-uncertainty**: *"Если не уверен — предложи списком, жди выбора. Не выбирай молча."*
- **Place philosophy at file edges** (top or bottom), not middle.

Apply this pattern when target is a literal model (any current Claude, GPT-5.x / 6.x) AND the prompt has deliberate hints-and-vibes framing. Skip if the prompt is already all-literal-rules.

**Non-prompt lever:** raising effort often restores hint-sensitivity better than any prompt edit (measured first on Opus 4.7 at `xhigh`; on Opus 5.5 the default dropped to `medium`, so check the setting first). Surface this as a conversation point before proposing a rewrite — never paste effort guidance into the file (SKILL.md § Reasoning-depth knobs).

Full snippet examples in `references/principles.md`.

---

## Pattern: creative-domain kernel

*Tuned on Opus 4.7 (now archived), whose literal, flatter defaults first made it necessary; the defaults it overrides persist in the literal Claude line, with the Opus 5-lineage caveats below.*

A close cousin of hint+anchor, but for a different problem. When the agent's **entire function** is creative / advisory / narrative / non-technical (game design, narrative, content, brand, strategy, coaching, brainstorming partner, design crit, fiction editor), Opus 4.7's defaults work against it on every turn: literalism kills inference, the flatter tone reads as cold, first-obvious-answer convergence kills divergent options, format-drift pushes prose into bullets.

The anti-pattern is to address this per-task — adding unlockers every turn. The correct move on a permanent-creative agent is to install a **kernel** at the system-prompt level (CLAUDE.md role section, subagent body, role-system text) that overrides Opus 4.7 defaults once.

#### Detection — when to apply

Triggers (at least two should match):
- Agent's primary output is ideas / options / critique / narrative / pitch / scenario
- Stated domain: game design, narrative, screenwriting, content, brand, marketing, strategy, coaching, therapy-adjacent, brainstorm, advisory, design
- User symptoms: "feels mechanical", "doesn't push back", "won't suggest things I didn't ask", "lost the warmth", "gives one obvious answer", "skips to bullets when I want prose"
- Russian cues: «сухо», «скучно», «без огонька», «не додумывает», «слишком технично», «как QA, не как дизайнер»

When prompt-atlas detects this case, the review should:
1. **First surface the non-prompt lever — ask the user about their effort setting, do NOT embed a recommendation in the file.** `low`/`medium` defeats any kernel. Advise out-of-band: for creative work default to `high`; `medium` for pure brainstorm; `xhigh` only when the creative task has a structured frame; `low` is almost always wrong here. Effort lives in the CLI / config / API call, not in CLAUDE.md or the kernel body — see § *Effort / reasoning_effort / thinking_level — surface, do NOT embed* above.
2. **Then propose installing the kernel** — 3–6 composable blocks from `techniques.md §25 "Whole-agent creative kernel"`, NOT all of them. Pick by domain.
3. **Always include the style anchor recommendation** — ask the user to paste 1–3 paragraphs in target voice. This is the single highest-leverage line on 4.7 for creative work.

#### Composable kernel blocks (full snippets in `techniques.md §25`)

Each line overrides a documented 4.7 default. Pick by which default is hurting the user:

| 4.7 default that conflicts | Kernel block to install |
|---|---|
| Obedient executor — literal interpretation | Role re-frame ("senior collaborator, not executor") |
| No inference / loss of "додумывание" | Expansion license ("treat request as minimum scope, raise adjacent moves") |
| Flatter, less validation-forward tone | Tone re-frame ("warm, direct, prose by default, no bullets/emoji/summary") |
| Checklist-clarify drift on ambiguity | Ambiguity handling ("interpret most likely meaning and proceed") |
| First-obvious-answer convergence | Default divergence ("3+ options from different roots, include a risky one") |
| Validation-forward agreement | Default pushback ("steelman opposition in one sentence before agreeing") |
| Habitual scaffolding ("I'd be happy to" etc.) | Anti-defaults block (named prohibitions) |

#### When NOT to apply

- Agent is mostly technical (coding, refactoring, ops) and only occasionally creative → use per-task unlockers (`techniques.md §25` top half) instead. The kernel's tone block will pollute regular technical output.
- Cross-vendor universal prompt — the tone block is Claude-shaped; Gemini and GPT react differently. Keep role re-frame + expansion license + divergence; drop tone.
- Haiku-tier targets — fewer abstract clauses, more concrete examples (measured on Haiku 4.5; Haiku 5.5 `?`).
- **Opus 5 / Opus 5.5 target — install selectively, two blocks now fight the model's defaults.** The kernel was tuned against 4.7's flatness, and Opus 5 no longer has the same defaults. **Expansion license** ("treat the request as minimum scope, raise adjacent moves") pushes in the same direction as Opus 5's own scope-widening — installing both produces an agent that reliably does more than asked. **Tone re-frame** ("prose by default, no bullets") compounds with defaults that already run long. Keep the blocks that still address real 4.7-era gaps — role re-frame, default divergence, default pushback, anti-defaults — and replace expansion license with the scope-boundary snippet (`techniques.md §20`) unless the user explicitly wants an agent that expands the brief. Verify against the symptom the user actually reported, not against the 4.7 default table.
- Agent has strong existing register that conflicts (formal legal advisor, compliance bot) — adapt tone, keep the rest.

**Prime directive interaction**: if the existing prompt already has lines that look like a kernel attempt — keep them, even if awkwardly worded. They're scar tissue from observed failures. Only add blocks that address gaps; don't rewrite an existing tone line unless the user asks.

---

## Pattern: small-model task prompt (Class 2)

Small local models on compile-time tasks live in a completely different prompt-engineering regime than frontier creative agents. This pattern is the small-model analog of the creative kernel — a standard layout that overrides the model's failure modes once at the structural level.

#### Detection — when to apply

All four should match:
- Target is a small local model — dense 2–9B or MoE ≤ ~9B active / ≤ ~35B total (Gemma 3/4, Qwen 3.5, Ministral 3, Phi-4-mini, Llama 3.2, Granite, MiniCPM5, LFM, Nemotron Dory, or a fine-tune)
- Output is structured (JSON object, JSON schema, DSL command, classification label) — not free-form prose
- Task is single-pass compile-time (extract / classify / NL→DSL / route / decide), not multi-turn conversation
- The prompt currently has abstract principles, hint-and-vibes prose, or no examples

#### What to install

The full skeleton lives in `techniques-small.md § Skeleton: small-model task prompt`. The non-negotiable structural elements:

1. **Task identity** in one neutral sentence at top (no persona block, no "You are an expert at...")
2. **Output section** before whitelist — shape, "no prose, no fence, no explanation"
3. **Whitelist / vocabulary** in a table
4. **Resolution procedure** in 3-5 numbered steps
5. **Forbidden** — 3-5 named anti-patterns, each one concrete failure mode
6. **Examples** at END — 4 for Gemma, 5 for Qwen, 4-5 for Ministral, mirroring failure classes
7. **Trailing "now read input below"** to prime the model

#### Family-specific adaptations

After installing the skeleton, layer on family-specific blocks from `models/small-local.md`:
- **Gemma**: no system role — fold instructions into first user turn; EN system on non-EN task
- **Mistral / Ministral**: markdown-tolerant output paragraph + named anti-patterns block ("CRITICAL: rules are sacred...")
- **Qwen**: anti-substitution block with 5 examples; always pass `enable_thinking=False`
- **Phi-4-mini**: avoid native tool calling without few-shot tool-call examples
- **Llama 3.2 / Hermes**: standard system role; JSON tool calling template

#### What NOT to install (small-model anti-patterns to strip)

Pre-existing prompt has any of these → flag as `[CRITICAL]` or `[IMPROVE]`:
- Abstract principles in body ("be careful with negations", "every adjective is mandatory")
- Hint-and-vibes prose copied from a frontier agent prompt
- Persona block ("You are an expert X with N years of experience")
- Long blacklist (>5 prohibitions, especially if no positive example follows)
- Same-model self-verify pass
- Native tool calling without few-shot tool-call examples
- System prompt in non-English when input is non-English and model is a base 2-3B (EN-system unlock applies)
- Thinking on / `reasoning_effort: high` for 2-3B (cite the −25 to −44pp regressions from `antipatterns-small.md § 2`)

Full inventory of 12 anti-patterns with reproduced regression sizes in `antipatterns-small.md`. Don't just flag from memory — cite the specific anti-pattern number so the user can audit.

#### When NOT to apply this pattern

- Target is a frontier model (Class 1) — use the frontier workflow path instead
- Target is a small local model but the task is free-form prose generation (chat companion, narrative continuation) — closer to a creative-kernel adaptation than a compile-time skeleton
- The user has a working benchmark suite showing the current prompt at >90% and is asking for marginal improvements — diminishing returns; recommend benchmark-driven iteration over more prompt edits

---

