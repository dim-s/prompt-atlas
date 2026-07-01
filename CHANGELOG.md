# Changelog

All notable changes to **prompt-atlas** are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/), and the project adheres to [Semantic Versioning](https://semver.org/) where feasible (model-coverage additions are minor versions; methodology changes are major).

## [1.3.0] — 2026-07-01

### Added

- **Claude Sonnet 5** (`claude-sonnet-5`, June 2026) — current Sonnet-tier frontier, demoting **Sonnet 4.6** to *previous* (still covered — it's the last Sonnet that accepts sampling tuning). Rows across matrix tables A–E, new `models/claude.md § Claude Sonnet 5` section, SKILL.md Step 2c routing + reasoning-knobs table + June-2026 updates block, `_universal.md` Universal-Claude, README coverage. Sourced from Anthropic's official "Prompting Claude Sonnet 5" and "What's new in Claude Sonnet 5" guides.
- **Z.ai GLM-5.2** (`zai-org/GLM-5.2`, released 2026-06-16) — current GLM frontier, demoting **GLM-5.1** to *previous*. Rows across matrix tables A–E, new `models/glm.md § GLM-5.2` section, SKILL.md routing + reasoning-knobs + updates, `_universal.md` Universal-GLM, README coverage. Sourced from Z.ai's official GLM-5.2 developer docs + independent benchmark reporting.
- **Subagent `effort:` frontmatter as a declarative effort-knob.** Documented Claude Code's official `.claude/agents/*.md` `effort:` field (`low`/`medium`/`high`/`xhigh`/`max`; **overrides session effort**) in the SKILL.md reasoning-knobs table and the "declarative-metadata exceptions" list — previously the list noted only Codex `model_reasoning_effort`, and the reasoning-knob table implied effort was CLI/API-only. Also clarified the prose-body antipattern to exclude frontmatter. Sourced from Claude Code sub-agents docs; surfaced during Sonnet 5 subagent-migration A/B testing.

### Review-relevant behavioral deltas — Claude Sonnet 5 (vs Sonnet 4.6)

- **Moved to Opus-level literalism.** The old "Sonnet is looser than Opus, generalizes more" model no longer holds — Sonnet 5 does not silently generalize scope. Prompts written for 4.6's inference under-apply; state scope explicitly. This changed the Table A literalism cell and the Universal-Claude "state scope" rule (previously Opus-only).
- **Sampling parameters removed** — `temperature` / `top_p` / `top_k` at non-default → 400, new for Sonnet-class (the constraint began on Opus 4.7). This propagated to a **cross-vendor rule change**: "don't reference temperature in the body" now covers the newest Claude *and* Gemini, not Gemini alone. Updated Table B temperature-gotcha (renamed from "Gemini-only"), the three-vendor and 4+ cross-vendor tables, and every temperature checklist item.
- **Adaptive thinking ON by default** (change from 4.6's thinking-off) + **new tokenizer (~30% more tokens)** — surfaced as migration/`max_tokens` notes, not prompt edits. Adaptive-thinking triggering is steerable from the prompt (snippet added).
- **More agentic** — readier tool use + self-verification loops; with thinking off it under-reaches for tools (add a nudge). **No subagent-spawn flip** (unlike Fable 5) — Table C marks it conservatively to avoid over-claiming.
- Verbosity calibrates to task, code-review harnesses need coverage language, frontend settles into a default style (propose-N-directions is the variety lever now that temperature is locked) — all mirror the Opus 4.7 snippets, cross-referenced rather than duplicated.

### Review-relevant behavioral deltas — GLM-5.2 (vs GLM-5.1)

- **Explicit `reasoning_effort` (`high`/`max`) parameter** — the headline wording change. Reasoning depth is now an out-of-band knob like every other frontier vendor. The `<reasoning_content>` prose re-injection workaround (family rule #1) drops to a **fallback** for routers that don't forward the param. Updated the SKILL.md reasoning-knobs table, the "exceptions where a knob lives in metadata" note, matrix Table B/C/E GLM cells, and the Universal-GLM reasoning rule.
- **1M lossless context** — genuine long-context capability, but it does **not** lift the <4 KiB `AGENTS.md` ceiling for router-mediated setups: host-prompt thinking-suppression is a reasoning-gate effect, not a context-length one. Called out explicitly to prevent a "big window = bloat the prompt" misread.
- **Identity-pinning still fails** (distillation artifact carried over); benchmarks (Terminal-Bench 2.1 81.0, SWE-bench Pro 62.1 > GPT-5.5, MCP-Atlas 77.0, FrontierSWE −1% vs Opus 4.8) and official prompting patterns (/goal mode, codebase-audit, standards-enforcement) added.

## [1.2.0] — 2026-06-11

### Added

- **Claude Fable 5** (`claude-fable-5`, released 2026-06-09) — new frontier tier above Opus; first public release of the Mythos line. Rows in matrix tables A–D, new `models/claude.md § Claude Fable 5` section, SKILL.md Step 2c routing + May–June updates block, README coverage. Sourced from Anthropic's official "Prompting Claude Fable 5" guide and "Introducing Claude Fable 5 and Claude Mythos 5".

### Review-relevant behavioral deltas (vs Opus 4.8)

- **`reasoning_extraction` refusal trigger** — "show / explain your reasoning in the answer" instructions in prompts and skills cause refusals with fallback to Opus 4.8. New highest-severity Fable-specific finding; added to universal-prompt checklist and scaffolding-to-strip list.
- **Subagent default flips** — Fable 5 delegates readily and sustains parallel/long-running subagents (Opus 4.8/4.7 undertrigger). Wording shifts from encouragement to boundaries; async orchestration preferred.
- **Over-prescriptive skills degrade output** — prior-model skills are often too prescriptive for Fable 5; burden of proof shifts toward trimming. Brief principle ≈ full enumeration (strong instruction following).
- **Long-run snippets** added: progress-grounding (anti-fabricated-status), action boundaries (anti-unrequested-actions), overplanning guard, early-stopping reminder, context-budget reassurance, intent-framing ("give the reason, not only the request"), memory hygiene, final-summary readability.
- Effort note (out-of-band, not embedded): `low` on Fable 5 often ≥ `xhigh` on prior Opus; adaptive thinking always on.
- **Narration flips quiet** (field observation, Jun 2026) — Fable 5 narrates less between tool calls than Opus 4.8; strip 4.8-era silence-default snippets (double-suppression), request update *shape* if visibility needed in interactive sessions.

## [1.1.0] — 2026-05-31

### Added

- **Claude Opus 4.8** (`claude-opus-4-8`, released 2026-05-28) as the current frontier Claude model across all five matrix tables (A–E), `models/claude.md`, `SKILL.md`, and cross-vendor comparison files. Opus 4.7 demoted to **previous** (still covered), 4.6 stays legacy.

### Changed (content-aware, not just version bumps)

- **Effort default** documented as `high` on all surfaces incl. Claude Code (4.8); effort flagged as "more important than any prior Opus."
- **Tool-triggering** cell flipped: 4.7 "undertriggers" → 4.8 "triggers required tools reliably; favors reasoning — raise effort/instruct for *more* tool use."
- **Adaptive thinking** note: 4.8 spends fewer thinking tokens than 4.7 at the same effort.
- New 4.8 API levers surfaced (not embedded): mid-conversation system messages, refusal `stop_details` categories, fast mode, 1,024-token cache minimum.
- Claude Code section notes **Workflows** (parallel-subagent research preview) and expanded **Auto mode**.
- **Creative-domain kernel** pattern renamed *for Opus 4.7 / 4.8* (same direct-tone / literal / convergent defaults persist); sourced to the new Prompting Opus 4.8 guide alongside the 4.7 migration guide.
- Cross-vendor benchmark refs updated where confident (e.g. SWE-Bench Pro: added Opus 4.8 = 69.2 alongside 4.7 = 64.3 in `deepseek.md`).

## [1.0.0] — 2026-05-25

### Added (initial public release)

**Frontier coverage — 9 vendor families:**
- Anthropic Claude (Opus 4.7 / Sonnet 4.6 / Haiku 4.5 + Opus 4.6 legacy)
- OpenAI GPT-5.x in Codex CLI (GPT-5.5 + Instant variant / 5.4 / 5.3 / 5.3-codex / 5.2 / 5.1)
- Google Gemini 3.x in Gemini CLI (3.1 Pro / 3 Flash / **3.5 Flash** / 3.1 Flash-Lite + 2.5 legacy)
- Moonshot Kimi (K2.6 / K2.5 / K2)
- Z.ai GLM (GLM-5.1 / GLM-5 / GLM-4.6)
- Alibaba Qwen frontier (Qwen3.7-Max / 3.7 Plus / 3.6 Plus / 3.6 Max-Preview / 3-Max-Thinking)
- DeepSeek (V4-Pro / V4-Flash / V3.2 / R1)
- xAI Grok (Grok 4.3)
- Mistral frontier (Mistral Large 3 / Mistral Small 4 / Ministral 3-8B+ reasoning)

**Small-local coverage:** Gemma 3/4, small Qwen 3.5 (2-9B), small Mistral/Ministral, Phi-4-mini, Llama 3.2, fine-tunes (saiga, T-lite, Hermes, HORROR-Imatrix, TrevorJS).

**Methodology — matrix-citation:**
- 5-table model × axis matrix (`references/matrix.md` for frontier, `references/matrix-small.md` for small-local)
- Every finding cites a specific row × column for auditability
- Cross-vendor compromise tables: 3-way (Claude + GPT + Gemini) and 4+ vendor (all current frontier)

**Vendor-specific anti-patterns documented:**
- GLM Claude-Code-router thinking suppression
- DeepSeek user-prompt-priority (opposite to every other vendor)
- Qwen un-emphasized section skipping
- Kimi Agent Swarms explicit-opt-in protocol
- Gemini 3.5 Flash silent regression — `thinking_level` default dropped from `high` to `medium`
- Identity-pinning failure on GLM (distillation artifact)
- DeepSeek V4 `reasoning_content` round-trip mandatory (breaking change from V3)

**Patterns documented:**
- Hint + literal anchor
- Creative-domain kernel for Opus 4.7
- Small-model task prompt skeleton
- Cross-vendor routers (Claude Code Router → GLM / Kimi / DeepSeek)

**Always-available:**
- `multilingual.md` — non-EN prompt handling, applies to both classes
- `principles.md` — universal principles across all vendors and classes
- `techniques.md` / `techniques-small.md` — copy-and-adapt snippets
- `antipatterns.md` / `antipatterns-small.md` — documented failure modes with fixes

### Source attribution

Each vendor model file ends with a "Source notes" section citing vendor docs, model cards, benchmark publications, and independent practitioner guides. Where independent verification is thin, axes are marked `?` rather than guessed.

---

## Version policy

- **Major (X.0.0)** — methodology change, breaking workflow change, file structure reorganization
- **Minor (x.Y.0)** — new vendor family added, new model variant added, new pattern documented
- **Patch (x.y.Z)** — fact updates (vendor publishes new guidance), citation corrections, typo fixes

When updating after a vendor releases a new model, bump the minor version and note in this changelog under the date of the update.
