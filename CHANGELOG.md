# Changelog

All notable changes to **prompt-atlas** are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/), and the project adheres to [Semantic Versioning](https://semver.org/) where feasible (model-coverage additions are minor versions; methodology changes are major).

## [1.9.0] — 2026-09-24

Coverage for the items found by the September 24, 2026 nightly reconciliation (`FINDINGS.md § Заход 2026-09-24`, BRIEFING.md «Что дальше» items 1–5), applied on the owner's approval of 24.09 (proposal `2f8679f465fbe1ab`). The vendor pages were re-read on 2026-09-24 (WebFetch summaries, not raw pages; numeric claims were cross-checked against a second page only where one existed — not for Qwen Omni-Flash or Sol / Luna) — the nightly report's own numbers were not copied.

### Added

- **Claude Opus 5.5** (2026-09-22, `claude-opus-5-5`) — new `models/claude.md § Claude Opus 5.5`, written as a delta on § Claude Opus 5 (Anthropic: existing Opus 5 prompts "should perform well without changes"). Thinking cannot be disabled (`thinking: disabled` / manual budgets and forced `tool_choice` → 400); effort default `medium`, and the same level thinks more than on Opus 5; the official end-of-system-prompt block for unattended runs that end a turn on a text report; progress-update levers; snippets for multi-app exploration, time budgets and pasted-content tags; chat "treat answered as done"; frontend named-pattern advice; safeguard notes (`reasoning_extraction`). The launch post's "40% less verbose" / "checks its own work" claims are recorded as vendor claims, **not** as prompt changes — the prompting guide changes no length or verification instruction. Matrix rows A, B, C, D; SKILL.md routing (options, updates, knobs row, gap-analysis row, the off-switch lists).
- **OpenAI GPT-6 Sol and GPT-6 Luna** (2026-09-22, `gpt-6-sol` / `gpt-6-luna`) — new `models/gpt.md § GPT-6 Sol and GPT-6 Luna`. **New addition, not a repair:** the reconciliation's claim that the atlas says "no `none`" without the Astra qualifier in six files did not hold up — all six places (`models/gpt.md`, `matrix.md`, `SKILL.md`, `agentic-systems/codex.md`, `agentic-systems/gemini-cli.md`, `artifacts.md`) were already narrowed to Astra, so nothing was corrected there. Documented differences from Astra: `none` supported; sampling parameters only at effort `none`; Chat Completions function calling only at `none`. The Astra behavior deltas are deliberately **not** ported: the guide documents them for Astra only. Sampling statements in `_universal.md` (two places), `SKILL.md` Step 4 and `agentic-systems/gemini-cli.md` gained "Sol / Luna: only at effort `none`". Matrix rows A, B, C, D.
- **DeepSeek V4.1 Flash** (2026-09-10, `deepseek-flash`; missed by the 1.8.0 window) — new `models/deepseek.md § DeepSeek V4.1 Flash`: new architecture, native vision, MIT; the ids `deepseek-v4-flash` / `deepseek-v4-flash-vision-exp` are temporarily routed to it; V4-Pro continues past 2026-09-14. The 08-19 / 08-21 field notes are marked as measured on V4-Flash, not re-run. Matrix rows A, B, D; SKILL.md routing.
- **Qwen3.8-Omni-Flash** (2026-09-18, `qwen3.8-omni-flash`, API-only) — new short `models/qwen-frontier.md § Qwen3.8-Omni-Flash`; omni input / text output, thinking on by default at `reasoning_effort` `xhigh`, `none` disables. **Secondary source only** (MarkTechPost quoting QwenCloud; the qwen.ai page did not render) — labelled as such in the file, the knobs row and the matrix.
- **xAI Grok 4.7** (2026-09-21, `grok-4.7`) — new `models/grok.md § Grok 4.7`: recommended model per xAI's models page, 500K, `reasoning_effort` low/medium/high/xhigh (default `high`), encrypted reasoning always returned on the Responses API. No prompting guide exists; no wording delta is invented. Matrix rows A, B (merged into the Grok family row), C.

### Changed

- **Status labels:** Claude Opus 5 → "previous Opus tier"; Grok 4.6 → "previous recommended" (`models/claude.md`, `models/grok.md`, `matrix.md`, SKILL.md options / updates / gap-analysis, README). Behavior statements about Opus 5 (verbosity, scope, subagents, thinking-off artifacts) are unchanged — they describe that model and remain the 5.5 baseline.
- **Class updates (L-0001):** the "no thinking off-switch" lists now include Grok 4.7 and Claude Opus 5.5 (SKILL.md, `_universal.md`, `antipatterns.md` #38, `matrix.md`); the Grok 500K-window statements cover 4.7; the "universal-Claude" row list in the matrix names Opus 5.5.
- README coverage lines and the `models/` tree.

### Decision recorded — DeepSeek `reasoning_effort`

The V4.1 Flash model card (Hugging Face) describes an integer 1–100 effort scale; DeepSeek's API thinking-mode guide documents strings only (`low` / `high` / `max`, plus `none` on the Anthropic surface, with a mapping of `minimal` / `medium` / `xhigh`). **The atlas rule stays** (`models/deepseek.md` rule #5); the integer is recorded as a property of the open weights, and whether the hosted API also accepts it is undocumented (`?`) — a prompt that sets an integer against the hosted API is a finding, not advice.

### Not verified / deliberately left out

- **Xiaomi MiMo-V2.6 (Pro / Flash / Distill-Qwen-9B)** — **not added.** A new vendor family needs an explicit owner decision (the atlas has no Xiaomi file); it is put to the owner as a proposal in `FINDINGS.md`. Facts read on 2026-09-24: release 2026-09-22, ids `mimo-v2.6-pro` / `-flash` / `-pro-ultraspeed`, 1M context; the release page did not state parameter counts, license or reasoning controls (the nightly report's 1.02T/42B, 309B/15B and MIT figures come from a secondary source and were not confirmed).
- **Sonnet 5.5 / Haiku 5.5 have not been released** as of 2026-09-24 (announced "in the coming weeks"); nothing was written for them.
- **openai.com launch post for Sol / Luna still returns 403**; coverage rests on the developer guide (read twice, consistent). The claim "Sol has half the errors of 5.6 Sol" from the press report was not carried in.
- **Grok 4.7:** the release post states neither the window nor the effort levels — those come from the models and reasoning docs pages.
- **Qwen3.8-Omni-Flash, DeepSeek V4.1 Flash image rules:** see above (`?`).
- **Not done:** matrix Table E rows for the new models (no documented persistent-context behavior); the open owner forks (Class 2 boundary, reconciliation rhythm, Glimmer placement, prices in sample sections) are untouched.

## [1.8.0] — 2026-09-13

Coverage for the four models found by the September 7, 2026 nightly reconciliation (BRIEFING.md «Что дальше» items 1–4), applied on the owner's approval of 13.09. Wording deltas are taken from the vendors' own documents, read on 2026-09-13; where a named page was unreachable, the substitute is another first-party document from the same vendor (see "Not verified").

### Added

- **Claude Fable 5.1 / Claude Mythos 5.1** (September 2026; `claude-fable-5-1` / `claude-mythos-5-1`) — successor to Fable 5 with an official prompting guide; Mythos 5.1 has the same capabilities and is limited to Project Glasswing participants. New `models/claude.md § Claude Fable 5.1 / Claude Mythos 5.1` covering the deltas that invert carried-over prompt content: formats less (strip anti-formatting rules), narrates less (remove "hold findings" lines, ask for an opening line + recap), one tool call per turn in coding / computer-use loops (per-turn batching nudge), the official autonomous-run block for async turns that end early or ask permission, the extras / test-scope snippet, and wording fixes for safeguard false positives (the Fable 5 section had none). Forced `tool_choice` now returns 400, so the prompt must say when a tool applies. Also: effort re-sweep, append-only history, system-card review notes; `reasoning_extraction` carried over by inference. Matrix rows A–D; SKILL.md routing (signals, options, updates, knobs label, gap-analysis row); README coverage.
- **OpenAI GPT-6 Astra** (`gpt-6-astra`; system card published 2026-09-03) — new frontier generation above GPT-5.6, with a latest-model guide. New `models/gpt.md § GPT-6 Astra`: collaborator default (asks and stops more readily — official initiative / approval-at-the-end snippets plus the system card's confirmation-policy result), heightened sensitivity to skills / `AGENTS.md` (audit + precedence line), detailed formatted default (inverts 5.6's terseness — prose and phrase-list snippets), less delegation (encouragement snippet — opposite of Claude Fable 5.1 / Opus 5), testing calibration, migration table from 5.6. API: `temperature` / `top_p` / `top_logprobs` unsupported, no `none` effort. Matrix rows A–E (E: Codex CLI with Astra); SKILL.md routing (description, signals, options, updates, knobs row, gap-analysis row); README coverage.
- **Google Gemini 3.8 Flash** (GA 2026-09-02, `gemini-3.8-flash`) and **Gemini 3.8 Flash Cyber** (closed access) — new `models/gemini.md` sections. 3.8 Flash works harder by design (extra reasoning steps, iterative tool calls, more tokens at higher effort), reversing the 3.6 / 3.7 terseness trend: don't port persistence scaffolding; efficiency is lower effort or 3.7 Flash. `thinking_level` low/medium/high, default `medium`, `minimal` → error. The Cyber variant goes to trusted defenders via the Fairwind Program (3.5 Flash Cyber went through CodeMender) — facts only. Matrix rows A–D; SKILL.md routing; README coverage.
- **Meta Muse Spark 1.3** (2026-09-02) — new `models/meta.md § Muse Spark 1.3`. First Muse Spark release with a documented behavioral delta: clarifying questions on ambiguous prompts, help-seeking when stuck, confirmation before consequential actions, update cadence adapted to user preference. Derived wording for interactive vs non-interactive runs (control surface undocumented — marked "test it"); other 1.2 → 1.3 deltas (long-instruction reliability, coding efficiency, prompt-injection robustness); max reasoning available, parameter name `?`. Matrix rows A–D and the Table E Meta row; SKILL.md routing (signals, options, updates, new knobs row with `?`, gap-analysis row); README coverage.

### Changed

- **Status labels** moved to "previous" where the new models supersede: GPT-5.6 (matrix Table A, SKILL.md options), Gemini 3.7 Flash (`models/gemini.md` heading, matrix Tables A/B, SKILL.md options), Muse Spark 1.2 (`models/meta.md` heading). Gemini family rules now read "3.0 → 3.8".
- **Sampling in cross-vendor summaries:** GPT-6 Astra noted as not accepting sampling parameters in `_universal.md` (three-vendor table, cross-vendor rule 6, 4+ strictest-source table), the `matrix.md` sampling gotcha, SKILL.md Step 4, and the Codex CLI cell of `agentic-systems/gemini-cli.md`.
- **Codex effort ladders without a version** (`agentic-systems/codex.md`, `agentic-systems/gemini-cli.md`, `artifacts.md`) note that GPT-6 Astra has no `none` (ladder `low` … `max`).
- Matrix reading guide: two new footnotes (the collaborator default of GPT-6 Astra / Muse Spark 1.3; "3.8 Flash works harder").

### Not verified / deliberately left out

- **The openai.com launch post and Safety overview for GPT-6 Astra returned HTTP 403** (WebFetch and curl) on 2026-09-13. Astra coverage rests on OpenAI's developer docs (latest-model guide, model page) and the GPT-6 Astra System Card on deploymentsafety.openai.com. The staged-rollout detail (approved organizations first, wider availability the next day) comes only from launch coverage and is not written into the atlas.
- **Mythos 5.1 is not generally available** (Project Glasswing only) — the nightly brief's "GA 01.09" applies to Fable 5.1.
- **`reasoning_extraction` is not mentioned in the Fable 5.1 docs** — carried over from Fable 5 by inference from the unchanged refusal categories, and labelled as inference.
- **Muse Spark 1.3's reasoning parameter and any control over the collaboration behavior** are undocumented — `?`.
- **Pricing and context-size figures** stay out of the new sections, per the model files' wording-only headers. Two existing sections still carry prices: Gemini 3.7 Flash and Qwen3.8-Flash. They were left as they are.
- **Pre-existing drift found, not fixed (outside this release's scope):** the `agentic-systems/gemini-cli.md` comparison table still says Gemini temperature "1.0 fixed" (deprecated 2026-07-21) and Claude Code "Tunable" (400 since Opus 4.7); effort ladders in `agentic-systems/codex.md`, `agentic-systems/gemini-cli.md`, `artifacts.md` and `techniques.md` lack GPT-5.6's `max`.
- **Follow-ups for a later pass:** the cross-vendor tables in `_universal.md` and `models/claude.md § Universal` don't yet carry the axes GPT-6 Astra and Fable 5.1 open up (formatting default, delegation propensity, clarifying-question propensity).
- **Owner forks untouched:** Class 2 boundary, reconciliation rhythm, Muse Glimmer 30B placement.

## [1.7.0] — 2026-08-29

Coverage catch-up for the August 19 → 29, 2026 window, driven by the nightly-reconciliation mandate (BRIEFING.md «Что дальше»). This pass also closes the long-open 1.6.0 item 7 (Meta Muse family update) and ships the pending housekeeping: `MANDATE.md` and `.BRIEFING.md.prev` are committed with this release; the branch is pushed in the same step (the repo had been one commit ahead of origin since 23.08). Every claim below was re-verified against the source links cited in `FINDINGS.md § Заход 2026-08-29` before being written.

### Added

- **DeepSeek V4-Flash-Vision-Exp** (2026-08-21, official changelog; `model='deepseek-v4-flash-vision-exp'`) — the DeepSeek family's first multimodal model, and the answer to the atlas's stale "V4 is text-only" line. Text on par with V4-Flash; vision-agent benchmarks close to Opus 4.8 (Terminal Bench 2.1 83.9, DeepSWE 59.3; beats Opus 4.8 on 3/11 per third-party coverage). **Two review-relevant facts:** images are accepted in **user messages only** (400 on system/assistant — the family's user-prompt-priority made structural), and ≤384 tokens per image after auto-resize (600 images/request max). Same effort-model thinking (`thinking` toggle + `reasoning_effort` low/high/max) as V4-Flash; `detail` low/original lever for cost-vs-fidelity. New `models/deepseek.md § DeepSeek V4-Flash-Vision-Exp` + fixed the V4-Pro "text-only" note + updated source notes; SKILL.md routing (signals / options / updates / knobs / cross-vendor axis / gap row).
- **Z.ai GLM-5.3-Flash** (2026-08-26; the former stealth `ox-alpha` that topped OpenRouter for six days) — first natively multimodal GLM-5: 320B-A18B, MIT weights (≈306 GiB FP8, Hopper+), $0.15/$0.50 per M with **cached input $0.03/M**. Same effort model as GLM-5.3 (`reasoning_effort` low/high/max, default `max`, no off-switch) + chat-template `clear_thinking` flag (default `false`). Caveats recorded: vendor's 1M-context claim coexists with 300K eval footnotes (treat 1M as a claim), vision is the admitted weak axis, benchmark sheet is vendor-owned. New `models/glm.md § GLM-5.3-Flash`; SKILL.md routing.
- **Alibaba Qwen3.8-Flash / Qwen3.8-Flash-Next** (2026-08-25/27) — cost-tier sibling of Qwen3.8-Max: 125B + 51B N-gram embedding, **6B active**, hybrid GDN+QSA attention (early Qwen4 architecture preview), multimodal; **262K native context on the open weights vs 1M default + built-in tools on hosted `qwen3.8-flash`** (QwenCloud) — the surface split is the review-critical fact. ¥1/¥3 per M (≈$0.16/$0.47); license `qwen-community-1.0` (custom, NOT Apache); SWE-bench Pro 62.5 vs DeepSeek-V4-Flash 56.0. Thinking knobs **unpublished** — marked `?` deliberately (anti-fabrication). New `models/qwen-frontier.md § Qwen3.8-Flash`; SKILL.md routing.
- **Meta Muse Spark 1.2 + Muse Code** (2026-08-05) and **Muse Glimmer** (2026-08-10) — closes 1.6.0's open item 7. Spark 1.2 keeps the same public-preview surface (chapter renamed, no prompting-relevant delta documented); Muse Code recorded facts-only (coding specialist, no documented behavioral delta); **Muse Glimmer 30B** is Meta's first open-weight agentic model (Apache 2.0, one consumer GPU, tool-use + failure recovery, controllable effort, multimodal input, OpenClaw compat) — new chapter + matrix rows A/B. `models/meta.md` updated; SKILL.md Meta signals/options renovated (the old "as of 1.1" is gone); matrix rows A–E renamed 1.1 → 1.2 (no residual in live cells); footnote "no prompting guide published" synced.
- **Tencent Hunyuan Hy4 preview** (2026-08-28) — **11th vendor family**. 770B/49B active, >1M context claimed, open weights; API via Tencent Cloud TokenHub + OpenRouter; $0.834/$2.501/$0.042 (cache); vendor's blind eval 2.99/4 vs GLM-5.3 2.92 / Kimi K3 2.94. **No prompting guide published** — new `models/tencent.md` is facts-plus-`?` by design (same treatment as Mistral Medium 3.5 / Muse). Matrix rows in all five tables (A–E) per the new-family rule; SKILL.md routing (description, Coverage, signals, options, Path A, auto-trigger, knobs row with `?`, gap row); README 11th family; `_universal.md` 4+ matrix entry.
- **IBM Granite 4.2** (2026-08-25) — first IBM family in the atlas, Class 2: 3B / 8B / 30B dense reasoning models, Apache 2.0 (30B above the 2-9B range, noted like Gemma 4 31B). **Switchable thinking in the chat template** (thinking default / non-thinking / low-effort — a serving choice, not prose); multi-turn thinking stripped by default; native tool calling in OpenAI format; 512K context (five-phase schedule); 8B trained with RL inside SE/terminal/search environments (unique for 2-9B). New `models/small-local.md § IBM Granite family`, rows in `matrix-small.md` tables A and B (untested cells `?`, no extrapolation), README Class 2 coverage.

### Changed

- **Matrix `Adding a new model` checklist** now carries the C–E norm explicitly: per-model rows in A/B, join family rows in C–E unless a documented difference exists (separate rows for undocumented deltas are fabrication); a brand-new family gets own rows in all five tables. Same norm mirrored in `CONTRIBUTING.md` short version.
- **SKILL.md Step 4 cross-vendor axes** gained the **image-content-placement** row (DeepSeek Vision-Exp: user-messages-only, 400 elsewhere) and the `_universal.md` 4+ matrix gained the same plus an anti-fabrication rule for guide-less new families.
- **Reasoning-depth knobs table**: GLM-5.3-Flash row (same effort model + `clear_thinking`), Qwen3.8-Flash row marked `?`, DeepSeek row extended to Vision-Exp, Tencent Hy4 row marked `?`.
- **DeepSeek V4-Pro "When NOT to invest"** — "Visual / multimodal — V4 is text-only" corrected to point at the Vision-Exp sibling.

### Not verified / deliberately left out

- **Gemini Omni 1.1 Flash** (GA 27.08) and **Gemini 3.5 Transcribe / Live** (GA 26.08) — media-generation / speech-to-text, outside the prompt-atlas scope; recorded in FINDINGS so a later pass doesn't re-investigate.
- **Mistral Shieldstral** already recorded in 1.6.0 as a guard-model niche; no change.
- **OpenAI GPT-5.6 in AWS Kiro** (24.08) — distribution news, no behavioral delta; recorded in FINDINGS.
- **No model-level releases found** for Anthropic (memory/Chrome GA are product news), Moonshot (K3 remains flagship), xAI (Grok 4.6 covered in 1.6.0), or Google's text-model line in this window.
- **Qwen3.8-Flash thinking knobs** and **GLM-5.3-Flash 1M context claim** — marked `?` in the atlas; both need vendor documentation.

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
