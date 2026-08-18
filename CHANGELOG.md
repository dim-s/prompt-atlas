# Changelog

All notable changes to **prompt-atlas** are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/), and the project adheres to [Semantic Versioning](https://semver.org/) where feasible (model-coverage additions are minor versions; methodology changes are major).

## [1.6.0] — 2026-08-18

Coverage catch-up for the July 28 → August 18, 2026 window, driven by the nightly-reconciliation mandate (BRIEFING.md «Что дальше», 18.08 pass). Items 1–6 and 8–10 of that pass are applied; item 7 (Muse Glimmer / Class-2 boundary), the owner-facing open forks, and leads L1–L4 remain open by design (item 7 and the forks need an owner decision; leads have no primary source per rule П10). Every claim below was re-verified against the source links cited in the briefing before being written.

### Added

- **Alibaba Qwen3.8-Max** (GA 03.08; open weights `Qwen3.8-2.4T-A95B` 12.08) — new frontier Qwen, closing the earlier П10 lead on the Max-Preview. First open Max-class release (2.4T / 95B active); hosted on **QwenCloud** (new host, OpenAI- and DashScope-compatible; $2/$6 per MTok, implicit cache $0.25 international) with hybrid thinking + vision input, while the open weights are text-only thinking-only (no off-switch). `reasoning_effort` low/medium/**xhigh** (default xhigh), `thinking_budget`, `preserve_thinking` (on by default), `/think` `/no_think` for open-source hybrid. 1M context with vendor-documented limits (input 991,808 / 983,616 thinking; output 131,072). New `models/qwen-frontier.md § Qwen3.8-Max`, SKILL.md routing (signals / options / updates / knobs table / gap-analysis row), README coverage.
- **xAI Grok 4.6** (Aug 12) — vendor's new recommended model ("for everything else, including code"). 500K context (the 4.3→4.5 regression did not revert), $2/$6, cached $0.50, double rate above 200K prompt, cutoff 2026-02-01, AA Intelligence Index 61 (parity with GPT-5.6 Sol). **Correction to stale atlas advice:** `reasoning_effort` is now documented for grok-4.5 and grok-4.6 (`low`/`medium`/`high` default `high`; `xhigh` on 4.6) — the old "no reasoning-effort parameter documented" line was wrong. Reasoning cannot be disabled on either model. New `models/grok.md § Grok 4.6`, SKILL.md routing + knobs table + gap-analysis row.
- **Z.ai GLM-5.3** (mid-Aug, launch post 17.08) — same base as 5.2, all gains from post-training (+50% Z.ai Code Bench; open-weight SOTA Terminal Bench 3.0 28.3 / Agents' Last Exam 28.5; emergent cyber CyberGym 84.5 / ExploitBench 54.4 — more than double 5.2). Text-only, 1M context, 128K output, weights ~2 weeks post-launch. **Third vendor (after Kimi) where thinking cannot be disabled:** `reasoning_effort` low/high/max (default max), `thinking.type` only accepts `enabled` — `disabled` requests fail. New `models/glm.md § GLM-5.3`, SKILL.md routing + knobs + gap-analysis row.
- **Google Gemini 3.7 Flash** (GA 13.08) — new Flash frontier three weeks after 3.6 Flash; default model powering the **Antigravity agent** and engine of **Gemini Spark**. 1M context, 64K output, `thinking_level` low/medium/high (**default medium**), intro price $0.75/$3.75 (half of 3.6 Flash; 3.6 dropped to the same rate) through 31.12.2026 then $1.50/$7.50; FrontierCode 43.6 vs 34.4, DeepSWE 65.3 vs 49.0, AutomationBench 30.4 vs 17.0 (vs 3.6 Flash). New `models/gemini.md § Gemini 3.7 Flash`, SKILL.md routing + knobs + gap-analysis row.
- **DeepSeek V4-Pro GA (0813)** — GA 13.08 under the unchanged `deepseek-v4-pro` name; agentic gains (HLE 42.7/60.0, Terminal-Bench 2.1 87.9, DeepSWE 62.7); native OpenAI Responses API support adapted for Codex. **Correction to stale atlas advice:** thinking is now an effort model — `thinking` toggle (`enabled`/`disabled`) + `reasoning_effort` low/high/max (default enabled, effort `high`; Anthropic surface `reasoning.effort` none/low/high/max; Responses API `output_config.effort`), replacing the old `off`/`high`/`max` triplet. Pricing: peak/off-peak from 16.08 (off-peak half), cache-hit tariffs up +1100%. Bonus: V4-Flash official release 31.07 + DeepSeek Harness v0.1 (open agent framework, MIT). Updated `models/deepseek.md` (family rule #5, V4-Pro / V4-Flash sections, source notes), SKILL.md routing + knobs.
- **OpenAI GPT-5.6 August update** (06.08) — improved Sol for Plus/Pro chat with an effort slider, Luna default for Free/Go with a Think button; successor to GPT-5.5 Instant's chat role; factual-error rate −68% (Sol) / −62% (Luna) vs GPT-5.5 Instant on high-stakes prompts; HealthBench Pro +15.6. **The version now depends on the surface:** ChatGPT = August; Codex and ChatGPT Work stay on July. New `models/gpt.md § August 2026 update`, SKILL.md gap-analysis row.
- **Gemma 4 stealth update** (2026-07-16) — Google re-published Gemma 4 weights under the same name: tool-calling fixes, truncated-response fixes, FA4 prefill acceleration on Hopper (H100+, +25–70% prompt throughput, −31% TTFT; Ada excluded from the speedup). Re-pull note added to `models/small-local.md § Gemma`.
- **Mistral Shieldstral** (04.08) — 3B open-weights multimodal safety classifier (policy-aware moderation, up to 7× smaller than comparable guard models); recorded as a guard-model niche note in `models/small-local.md § Mistral`. Not a prompt target — documented so a later pass doesn't re-investigate.

### Changed

- **Reasoning-depth knobs table (SKILL.md) updated in four cells** (brief item 10): Grok gained `reasoning_effort`; DeepSeek moved to the effort model (low/high/max, default high); GLM-5.3 is a new vendor with thinking forced on (row now Kimi + GLM-5.3, no longer kimi-specific); Gemini 3.7 Flash row notes default `medium`. Qwen3.8-Max row added. Cross-vendor rule: of nine vendors, three now have thinking that cannot be disabled or is enabled by default with effort levels — "не думай" phrasing loses meaning wider than antipattern #38's original scope.
- **Step 4 contradiction lists** — "answer without reasoning" and the thinking off-switch axes now cover Kimi K3/K2.7-Code, GLM-5.3, Grok 4.5/4.6, and the Qwen3.8 open weights.
- **GPT-5.6 surface-split** added to the SKILL.md updates block and gap-analysis (Chat = August, Codex/Work = July).

### Not verified / deliberately left out

- **Ultrafast mode** (brief item 6's "limited preview, Sol up to 14× faster") was not written into the atlas — the item's own linked sources (openai.com index post, deployment-safety page, help-center release notes) do not mention it, so per the quality gate it stays out pending a next-pass re-check.
- Item 7 (Muse Glimmer), the two owner-facing open forks (Class-2 boundary, reconciliation schedule), and leads L1–L4 remain untouched and are tracked in BRIEFING.md.

## [1.5.0] — 2026-07-28

Coverage catch-up for the May–July 2026 window, driven by the model-watch mandate: the findings board (`FINDINGS.md`, 2026-07-28 pass) is now worked through. Every claim below was re-verified against the vendor's own page before being written into the atlas, and two claims from the findings board were **corrected** in the process (noted under *Not verified / corrected*).

### Added

- **OpenAI GPT-5.6 — Sol / Terra / Luna** (`gpt-5.6-sol` / `-terra` / `-luna`, bare `gpt-5.6` aliases Sol; public July 9, 2026). A whole missing family — the atlas stopped at 5.5. New `models/gpt.md § GPT-5.6`, rows across matrix tables A–E, SKILL.md Step 2b/2c routing + updates block + a gap-analysis row, `_universal.md` Universal-GPT rules, README coverage. Model-card facts: 1,050,000-token context (input ≤922,000, output ≤128,000), knowledge cutoff 2026-02-16. Sourced from OpenAI's latest-model guide and model card.
- **Moonshot Kimi K3** (API July 16, open weights July 26, 2026) and **Kimi K2.7-Code** (June 12, 2026). Two missed releases in a row. New `models/kimi.md § Kimi K3` and `§ Kimi K2.7-Code`, matrix rows A–E (plus a Kimi row in table E), family-rule rewrite for the thinking toggle, `_universal.md` Universal-Kimi, README coverage. Sourced from the HuggingFace model cards.
- **Google Gemini 3.6 Flash** and **Gemini 3.5 Flash-Lite** (both GA July 21, 2026), plus a note on **3.5 Flash Cyber** as closed-access and therefore untunable. New sections in `models/gemini.md`, matrix rows A–E, README coverage. Sourced from the Gemini API changelog.
- **xAI Grok 4.5** (July 2026) — now the vendor's recommended model. New `models/grok.md § Grok 4.5`, matrix rows A–E including a table-E row for the xAI API, gap-analysis row. Sourced from xAI's models page.
- **Meta Muse Spark 1.1** (July 9, 2026) — **new vendor file** `models/meta.md`. Meta moved from "closed preview, most axes `?`" to an addressable API vendor (public-preview Meta Model API, OpenAI-compatible, 1M self-managed context, documented planning mode / goal conditioning / subagent delegation / context compaction). SKILL.md Step 3 now routes to the file instead of saying none exists; README lists Meta as the 10th vendor family. Behavioral axes stay `?` — see below.
- **Mistral Medium 3.5** (`mistral-medium-3-5-26-04`, April 28, 2026) — new section in `models/mistral-frontier.md` plus matrix rows. Facts only, by design.

### Changed — the July 2026 cross-vendor shifts

- **Sampling-parameter lockout is now a cross-vendor trend, not a Claude quirk.** Google **deprecated `temperature` / `top_p` / `top_k` API-wide on 2026-07-21**, joining Anthropic (non-default → 400 from Opus 4.7 onward). The matrix's "Temperature gotcha" is rewritten as a **sampling gotcha** covering both vendors; `models/gemini.md` family rule #3 changes from "don't tune" to "deprecated"; the three-vendor and 4+ cross-vendor tables, every temperature checklist item, and the Step 4 contradiction list follow. Practical consequence stated once and reused: tone and variety are a wording problem now, and "propose N directions" replaces the knob.
- **Response-length defaults now contradict across vendors — new Step 4 contradiction axis.** GPT-5.6 and Gemini 3.6 Flash are *terser* than the versions they replace (a carried-over "be brief" overcorrects), while Opus 5 runs long and doesn't calibrate (concision must be prompted, `effort` won't fix it). The atlas previously carried only the Claude half. New guidance, stated in `techniques.md §19`, `models/gpt.md`, `models/gemini.md`, matrix reading guide, and `_universal.md`: **express length as a requirement of the deliverable, never as a disposition of the assistant.**
- **Instruction repetition became a measured cost.** OpenAI reports leaner system prompts scoring ~10–15% higher while using 41–66% fewer tokens on 5.6. On a 5.6-targeted review, deleting duplicated instructions and verbose tool descriptions outranks rewriting them — the gap-analysis row for GPT-5.6 says so explicitly ("expect strips to outnumber adds", for the opposite reason to Opus 5's).
- **Anti-pattern #38 ("telling the model not to think") gained a third failure mode.** On **Kimi K3 and K2.7-Code the instruction cannot be honored at all** — thinking is forced on with no disabled mode. First family in the atlas without an off-switch, which turns the pattern from target-dependent into an unconditional strip. Mirrored in the matrix reasoning-depth rule and the 4+ compromise matrix.
- **Grok's context window regressed on upgrade: 1M (4.3) → 500K (4.5).** Rare enough that the atlas had no warning for it. Recorded as a `[CRITICAL]`-class migration finding wherever the prompt architecture assumes 1M, plus the **200K pricing step** that gives accumulated persistent context a hard cost boundary on xAI.
- **Gemini 3.5 Flash-Lite is the first vendor-designated subagent model** — Google's own changelog wording. Documented in the subagent-relevant cells (matrix table C) and in `models/gemini.md`, alongside the cross-vendor observation that Haiku 4.5 / Flash-Lite / GPT-5.6 Luna are interchangeable in that role modulo the format-contract syntax.
- **DeepSeek legacy aliases removed from the live vendor signals.** `deepseek-chat` / `deepseek-reasoner` were discontinued 2026-07-24 per DeepSeek's official changelog; SKILL.md Step 2b now treats them as a migration finding (dead endpoint) rather than as an indicator of which current model is in use. Small edit, but it was producing wrong advice as of the day it shipped.

### Not verified / corrected

- **Kimi K2.7-Code's system prompt is illustrative, not mandated.** The findings board recorded the vendor as "dictating a literal system prompt". The card shows `You are Kimi, an AI assistant created by Moonshot AI.` in a chat example and states no requirement — `models/kimi.md` says so explicitly, and the identity-pinning section was not rewritten on the strength of an example.
- **Gemini 3.5 Flash Cyber is not in the API changelog** — it appears in the launch blog post only, with access limited to governments and trusted partners via CodeMender. Recorded as existing-but-untunable rather than as a covered model.
- **No prompting guidance exists for Grok 4.5, Mistral Medium 3.5, Muse Spark 1.1, or the two new Gemini Flash models.** Their cells inherit family defaults and their sections say so. The absence is written down deliberately so a later coverage pass doesn't re-investigate — and so no version-specific wording delta gets invented to fill the gap.
- **Muse Spark's OpenAI-compatibility / structured-output / parallel-tool-call wording comes from an early-partner testimonial** in Meta's post, not from Meta's specification text. Treated as an availability claim, not a behavioral guarantee.
- **Alibaba Qwen3.8-Max-Preview was deliberately not added.** Announced July 19 at WAIC, but there is no model card, no benchmark table, no price and no license, and it isn't listed in Alibaba Cloud Model Studio. Held as a lead in `FINDINGS.md § П10`.
- **The Class 2 boundary question (`2-9B` vs `≤~9B active parameters`) is left open.** MoE models like `Qwen3.6-35B-A3B` and `Gemma 4 26B-A4B` behave like small local models by active parameters and hardware, but fall outside the class as currently defined. That's a redefinition of the skill's frame, not a coverage gap — recorded in `FINDINGS.md § П8` for the owner to decide.

## [1.4.0] — 2026-07-25

### Added

- **Claude Opus 5** (`claude-opus-5`, July 2026) — current Opus tier and Anthropic's default recommendation for complex agentic coding, demoting **Opus 4.8** and **Opus 4.7** to the legacy table. Rows across matrix tables A–E, new `models/claude.md § Claude Opus 5` section, SKILL.md Step 2c routing + July-2026 updates block + gap-analysis row, `_universal.md` Universal-Claude, README coverage. Sourced from Anthropic's official "Prompting Claude Opus 5", "What's new in Claude Opus 5", "Prompting best practices", "Effort", and migration guide.
- **Anti-pattern #37 — carried-over self-verification instructions on Opus 5.** The Claude-family analog of #36 (paying for behavior the model already has). Documents the strip-don't-soften rule and the line between task verification (keep) and self-review (remove).
- **Anti-pattern #38 — telling the model not to think.** Inert as a reasoning control everywhere; on Opus 5 with thinking disabled it *increases* internal-XML-tag leakage into visible output. Includes the official combined mitigation for text-form tool calls and leaked tags.
- **Within-family contradiction table in SKILL.md Step 4.** New section: as of July 2026 the Claude family disagrees with itself on self-verification, subagent delegation, and narration — so "it's a Claude prompt" no longer determines the recommendation. Step 2c now instructs asking for the Claude *version* even when the vendor is obvious.
- **`models/claude.md § Universal` rule 7** — divergence table plus conditional phrasings that survive the whole family (state the *condition*, not the direction).

### Changed — inverted advice (four Opus 5 deltas that reverse prior guidance)

This is the largest set of reversals in any Claude release the skill has tracked. Each was correct guidance on Opus 4.7/4.8 and is now wrong on Opus 5:

- **Self-verification: `[ADD]` → `[CRITICAL]` strip.** Opus 5 verifies its own work unprompted; carried-over verification instructions cause over-verification with no quality gain, and Anthropic's guidance is removal, not rewording. `techniques.md §20` gained an explicit Opus-5 exception, and the SKILL.md gap-analysis row for tool-using subagents — which recommended adding a verification step unconditionally — now carves Opus 5 out. Highest-impact change in this release: the skill was actively recommending the harmful pattern.
- **Verbosity: "calibrates to task" → does not calibrate.** Every other current Claude shortens simple answers on its own. Opus 5's defaults run long, and `effort` controls thinking volume, not visible length — so the standing advice "raise/lower effort before rewriting" is wrong for verbosity complaints specifically. Documented in the SKILL.md reasoning-knobs section, matrix table B, and `techniques.md §19`, which also gained a separate length-calibration snippet for **written files** (a new axis — deliverables run long independently of chat).
- **Subagents: encouragement → boundaries + cap.** Opus 5 flips to readily delegating (like Fable 5), inverting the 4.7/4.8 undertriggering default that `techniques.md §17`'s encouragement snippet was written for. Added the damping snippet alongside it.
- **Narration: quiet → talkative.** Opus 5 narrates more between tool calls, the opposite of Fable 5's field-observed quiet. 4.8-era silence-defaults, which `models/claude.md` told reviewers to strip for Fable 5, stay useful here — a shared Claude prompt cannot carry one narration setting.

### Changed — other

- **Creative-domain kernel gained an Opus 5 caveat.** The kernel was tuned against Opus 4.7's flatness; on Opus 5 two of its blocks fight the model's own defaults — *expansion license* compounds with Opus 5's scope-widening, and the *tone re-frame* compounds with already-long output. Install selectively.
- **Scope guidance widened.** Prior Opus models only failed to generalize *downward*; Opus 5 can also add unrequested steps, so narrow tasks need both boundaries stated. New snippet in `techniques.md §20`.
- **Code-review coverage language reinforced.** Opus 5 finds real bugs at a high rate with few false positives, so "only report high-severity issues" discards good findings; accuracy holding at low effort makes a cheap-pass/thorough-pass harness viable.
- **Table E context-rot caveat.** Opus 5's instruction following and tool calling are documented as consistent across its full 1M window — the ~300-line rot heuristic doesn't apply to it (and shouldn't be generalized from it).
- `models/claude.md` Sonnet 4.6 section pointed at a non-existent "Opus 4.8 section"; the Opus 4.7 section is now explicitly labelled as covering 4.8, and the cross-reference is fixed.

### Not verified

- **Sampling-parameter rejection on Opus 5.** The `temperature`/`top_p`/`top_k` → 400 constraint is not restated in the Opus 5 docs and is not listed among its breaking changes from 4.8, so it's recorded as carried over — **an inference from absence, not a quoted statement**. Flagged as such in `models/claude.md`, matrix table B, and `_universal.md` rather than asserted.

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
