# Archive — legacy sections from `models/gpt.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

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

## GPT-5.1 (legacy)

Earliest GPT-5 variant. Most current tooling has migrated past it; you'll mostly see 5.1 prompts in archived templates or older internal docs.

### Behaviors that shape wording

- **Less reasoning depth**: prompts that work on 5.1 often look heavily scaffolded with explicit step plans. That's a 5.1 adaptation, not a universal best practice.
- **Higher hallucination rate vs newer family**: anti-hallucination snippets (`../techniques.md` §12, §22) carry their weight here.
- **Shorter effective context**: long-document prompts that work on 5.5 may not fit cleanly on 5.1.

### When the user's prompt is for 5.1

- The prompt is probably overdue for a rewrite, but don't push if the user has a reason to stay on 5.1 (cost, latency, stability, internal policy).
- Treat 5.1 wording as a record of the model's quirks at that point — many of the patches the user added were answers to specific failures, even if they look stale by 2026 standards. Apply the **Prime directive** in SKILL.md.
