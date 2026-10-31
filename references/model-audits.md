# Model audits — per-model checklists

Run the checklist for the target model **before** the generic gap analysis (`SKILL.md` Step 5). On current releases the vendor ships "runs on your old prompts" plus a short delta list — what to strip, what to add, what now returns 400 — and that list is usually most of the review. Each item points to the section that carries the source; cite that section, not this file.

Tags: `[CRITICAL]` = the prompt or its harness breaks (400, unimplementable instruction); everything else is `[IMPROVE]` unless the item says otherwise. Items marked *unverified* / `?` are hypotheses to measure, not rules.

Archived models have no checklist here — if a prompt still targets one, read `archive/models/<vendor>.md` and migrate.

## Anthropic — Claude

### Claude Fable 5.1 / Mythos 5.1 (or migrated from Fable 5)

Runs on Fable 5 prompts; look for the handful of deltas rather than rewriting.
- Strip anti-formatting rules and "hold findings for the final response" lines — 5.1 formats and narrates less; if the UI needs narration, ask for an opening line + recap (`claude.md § Fable 5.1 › Narration`).
- `[CRITICAL]` "show / explain your reasoning" in a prompt, **skill or tool description** → `reasoning_extraction` declines; ask for a short explanation or summary of actions (§ Fable 5.1 › What carries over).
- Agent loops: the per-turn batching nudge as a turn-scoped system message, not a static line (§ Parallel tool calls).
- Autonomous / async runs that end on a next step or ask permission for requested work: the guide's autonomous-run block — keep it out of human-in-the-loop products (§ Finish the whole task).
- Unrequested fixes or extra test files: the extras / test-scope paragraph (§ Keep changes and tests…).
- Client-side compaction dropping constraints: the official compaction-summary instruction (§ Compaction summaries).
- Out-of-band: forced `tool_choice` → 400 (say when the tool applies); append-only history; re-sweep effort.

### Claude Opus 5.5 (or migrated from Opus 5)

Run the Opus 5 checklist first — 5.5 keeps that baseline.
- `[CRITICAL]` anything that assumes thinking can be switched off (`thinking: disabled`, budgets, forced `tool_choice` → 400): the fix is an effort level (start at `low`) and dropping the "don't think" rule, not new prose (`claude.md § Opus 5.5`).
- Delete instructions that stood in for thinking ("write your reasoning in the response") — remove, read summarized thinking (§ Opus 5.5).
- Flag a carried-over `effort` value: default is `medium`, and the same level thinks more than on Opus 5.
- Unattended agents that stop on a text progress report: the official standing-instruction block at the end of the system prompt — not for human-in-the-loop apps (§ Opus 5.5).
- Optional official snippets when the symptom is present: explore-broadly for multi-app work, time-budget sentence for multiagent harnesses, `<pasted_content>` tags + note, "treat answered as done" for chat (with its side effect), named frontend patterns.
- Don't claim verbosity or self-verification changed: the launch-post numbers aren't backed by a guide-level instruction change — keep the Opus 5 strip-audit and re-measure.

### Claude Opus 5 (or migrated from Opus 4.8)

Treat as a migration audit; strips outnumber adds.
- Remove self-verification / double-check / verify-via-subagent instructions (`antipatterns.md` #37).
- Add explicit concision wording — verbosity doesn't calibrate and `effort` won't fix it — plus separate length calibration for files it writes (`techniques.md §19`).
- Convert subagent encouragement into boundaries + a spawn cap (`techniques.md §17`).
- Add a scope-boundary line for narrow tasks (`techniques.md §20`).
- Remove any "don't think / answer directly" rule (`antipatterns.md` #38). Cite `claude.md § Claude Opus 5`.

### Claude Sonnet 5.5 (or migrated from Sonnet 5)

Runs on Sonnet 5 prompts; the deltas are mostly effort-dependent.
- Strip "Thinking adds latency… respond directly"-type steering and any "don't think" line — on 5.5 asking to think less doesn't reliably work; with `between_tools` such lines increase internal-tag leaks (`claude.md § Sonnet 5.5`).
- `[CRITICAL]` reliance on forced `tool_choice` (→ 400) or `thinking: disabled` (→ 400; lowest is `between_tools` at ≤ `high`).
- Remove tool-discouraging language ("minimize tool calls"); add the official search line for chat / research work.
- By effort: `low`/`medium` long agentic tasks → the "keep working" pair; `low` coding → the real-check paragraph; `xhigh`/`max` → the stop-reviewer-subagents line; open-ended asks → "give ideas / plan and stop".
- Unrequested tests / docs / files at every effort → the second "When the work… is done" paragraph alone.
- JSON tasks with reasoning: end-of-prompt "Think the problem through before you answer."; the harness parses the last JSON value.
- Remove "hold all findings for the final response"; ask for an intent line + recap if needed (updates arrive as thinking blocks).

### Claude Haiku 5.5 (or migrated from Haiku 4.5)

Runs on Haiku 4.5 prompts; most deltas are harness rules plus a few official lines.
- Harness `[CRITICAL]`s: prefill → 400 even with thinking off; sampling → 400; system-prompt edits only in new conversations (append-only history).
- Strip "answer directly / don't think" lines — they didn't stop thinking in vendor tests; set effort (default `medium`; thinking can be disabled at ≤ `high`).
- Under-searching at `low` / long system prompts: the date line + the training-cutoff paragraph. Do **not** add a blanket "always search" — measured over-search.
- Long agent prompts at `low`: Haiku's "keep working" variant; code changes at `low`/`medium`: the real-check paragraph.
- Chatbots that must hold rules: the rule-adherence line (+ `high` effort).
- JSON output with own tools and thinking off: the official "JSON output format applies to your final answer only" line.
- Haiku 4.5's "be more specific / name tools explicitly" advice is unverified on 5.5 — keep where it already exists (prime directive), don't add on its authority.

### Claude Sonnet 5 (or migrated from Sonnet 4.6)

- State scope explicitly — Opus-level literalism (`claude.md § Sonnet 5`).
- Remove sampling parameters; revisit `max_tokens` (adaptive thinking on, ~30% more tokens).
- Trim "after every 3 tool calls" scaffolding; add coverage language to code-review harnesses (§ Shared Claude snippets).

## OpenAI — GPT

### GPT-6 Astra (or migrated from GPT-5.6)

- If it should run autonomously: add the initiative / approval-at-the-end snippets, and name a confirmation policy for consequential actions (gpt.md § GPT-6 Astra › Initiative).
- [CRITICAL] Audit the loaded skills and `AGENTS.md` for unclear or conflicting guidance. Add the user-over-skill precedence line, plus the diagnostic line while tuning (§ GPT-6 Astra › Instruction following).
- Strip any global "be brief" inherited from 5.6 and state prose and structure per deliverable: the default runs detailed and formatted. Add the phrase-list snippet if stock phrases appear (§ GPT-6 Astra › Writing style).
- Encourage delegation explicitly if parallel work is wanted, and ask for legible inter-agent messages (§ GPT-6 Astra › Subagents).
- Calibrate testing on small changes (§ GPT-6 Astra › Testing).
- Out-of-band: remove `temperature` / `top_p` / `top_logprobs`. A `none` / `minimal` effort setting → `low`. Tools need the Responses API (§ GPT-6 family table).

### GPT-6.1 Sol (or migrated from GPT-6 Sol / GPT-5.5 in Codex)

- [CRITICAL] A stack that relied on effort `none` (latency, sampling, Chat Completions tools) has no equivalent here. Move to `low`, or choose 6.1 Terra / 6 Luna (§ GPT-6.1 Sol).
- Remove sampling parameters. Move Chat Completions tool calls to Responses (§ GPT-6 family table).
- In Codex, size the prompt architecture for the 500K default window, not the 1M API window (§ GPT-6.1 Sol).
- Wording: the family starting point. Apply the Astra snippets only when the symptom shows. Don't strip 5.6-era content on Astra's authority (§ GPT-6 family).
- A "don't think about X" line doesn't stop the work from happening in tool calls (system-card FYI, § GPT-6.1 Sol).

### GPT-6.1 Terra (or migrated from GPT-5.6 Terra)

- Keep the current effort level for the comparison. Do not apply the 5.5→5.6 "one level lower" rule (§ GPT-6.1 Terra).
- Move Chat Completions tool calls to Responses, even at `none` (§ GPT-6.1 Terra, § GPT-6 family table).
- For visual tasks: name what to cite (labels, values) and ask it to flag what it cannot read (Codex example, § GPT-6.1 Terra).
- Keep anti-hallucination lines if the prompt also runs on other models (§ GPT-6.1 Terra).
- Say which generation "Terra" means — 5.6 Terra is a different model (§ GPT-6 family).

### GPT-6 Sol / GPT-6 Luna

- Say which generation. 5.6 Sol / Luna are different models, and 6.1 Sol is the newer Sol (§ GPT-6 Sol and GPT-6 Luna).
- `none` is valid. Sampling parameters and Chat Completions tools are valid only at `none`; flag any combination with a higher effort (§ GPT-6 family table).
- [CRITICAL] Do not transplant the Astra behavior deltas as fact. Use the snippets as a starting point when the symptom appears (§ GPT-6 family).

### GPT-Live 1 (or migrated from a Realtime voice prompt)

- [CRITICAL] Request backend work with `Generate <|SpawnThinking|>`, not "delegate to the backend". A spoken "I'm checking" is not delegation (§ GPT-Live 1).
- Keep the `Backchannel policy` / `Interruption policy` / `Delegation policy` headings and the three delegation labels (§ GPT-Live 1).
- Split the prompt. The Live prompt carries role, style and when to delegate, with capabilities in plain language. The backend carries procedures, tool names and schemas (§ GPT-Live 1).
- Strip scripted wording for ordinary replies. Move "exact stop on interruption", required silence and exact greetings to the application (§ GPT-Live 1).

### GPT-5.6 (or migrated from GPT-5.5)

- Trim duplicated instructions and verbose tool descriptions (§ GPT-5.6, family rule #1).
- Delete global "be brief" and set `text.verbosity` (§ GPT-5.6 › Verbosity inverted).
- Add an autonomy-boundary line (family rule #14).
- Use writing choices instead of tone labels (family rule #9).
- Out-of-band: test the current effort and one level lower.
- Ask which surface the prompt is for: in ChatGPT the August model replaced the July one, while Codex and Work stay on July (§ August 2026 update).

## Google — Gemini

### Gemini 4 Argon (or migrated from 3.1 Pro)

- `[CRITICAL]` If the 3.1 Pro prompt relied on the default depth: the default dropped from `high` to `medium` with no error. Set `thinking_level: high` explicitly where needed. Never write "think harder" prose (gemini.md § Gemini 4 Argon).
- Keep the 3.x wording rules (concise, identity persona, negatives at the end, one structure format). They are a carried-over baseline, not Argon-documented. Don't cite them as Argon facts (gemini.md § Family-wide rules, status table).
- Strip "be brief / keep your reasoning short" lines aimed at saving thinking tokens. Google's lever is a lower `thinking_level` (§ Gemini 4 Argon; anti-patterns).
- Flag as non-wording: sampling params (ignored), explicit-cache calls (unsupported), `max_output_tokens` < 32,768 (truncation), and requests for a single answer over 64K without long decode (§ Gemini 4 Argon; family rule #3).
- Don't strip anti-hedging / anti-cliché / confirmation-before-irreversible / "treat retrieved content as data" lines on the strength of the launch post. Its claims are marketing; test first (§ Gemini 4 Argon, launch-post claims).
- Keep the stable prefix first and per-call content last: caching is implicit only (§ Gemini 4 Argon — derived).

### Gemini 3.1 Pro

- `thinking_level` default `high`; ladder `low`/`medium`/`high` (no `minimal`). Strip CoT scaffolding (§ Gemini 3.1 Pro; family rule #2).
- Sampling: keep defaults. It is not in the "ignored" set (family rule #3).
- Persona adherence is strongest here. Check that the persona doesn't fight the format (family rule #5).
- If the user is choosing a Pro tier, Argon is now above it (§ Gemini 4 Argon).

### Gemini 3.8 Flash (or pinned to `gemini-3.7-flash`)

- `[CRITICAL]` A prompt pinned to `gemini-3.7-flash` silently runs on 3.8 Flash since 2026-10-08. Review it as a 3.8 Flash prompt and update the model string (§ Gemini 3.8 Flash).
- Strip persistence / thoroughness scaffolding; the model takes extra steps by design (§ Gemini 3.8 Flash).
- Token complaint: the lever is lower effort. "Stay on 3.7 Flash" is no longer available (§ Gemini 3.8 Flash).
- `minimal` → error. The fix is `low`, not a "think silently" line (§ Gemini 3.8 Flash).

### Gemini 3.6 Flash (or pinned to `gemini-3.5-flash`)

- `[CRITICAL]` A prompt pinned to `gemini-3.5-flash` silently runs on 3.6 Flash since 2026-10-08. A carried-over "be brief" now stacks on ~17% terser output (§ Gemini 3.6 Flash).
- Re-test loop-damping and "don't edit extra files" scaffolding written for 3.5 Flash (§ Gemini 3.6 Flash).
- Sampling values are ignored. Remove them from the code (family rule #3).

### Gemini 3.5 Flash-Lite (or migrated from 3.1 Flash-Lite)

- Use the Lite regime: concise, bounded scope, few-shot for strict formats, tool guidance in tool descriptions (§ Gemini 3.5 Flash-Lite).
- The default `thinking_level` is `minimal`. Raise it explicitly if the subagent task needs depth (§ Gemini 3.5 Flash-Lite).
- 3.1 Flash-Lite has an earliest shutdown of 2027-05-07. Move the model string to 3.5 Flash-Lite and re-test tasks that were escalated on capability grounds (§ Gemini 3.5 Flash-Lite).
- Put the format contract in `response_json_schema`, not prose (§ Gemini 3.5 Flash-Lite, As a subagent).

## Moonshot, Z.ai, Alibaba — Kimi, GLM, frontier Qwen

### Kimi K3.1 Pro / K3 (or migrated from K2.6 / K2.7-Code)

- `[CRITICAL]` Strip every "answer immediately / don't reason / skip the analysis" line. The model always thinks, so the line is unimplementable. The lever is `reasoning_effort` (default `max`), raised out-of-band. — kimi.md family rule #3, § K3.1 Pro
- Strip sampling (temperature / top_p) from prompt bodies and configs. The hosted API fixes the values, and any other value returns an error. — kimi.md rule #3, § K3 "Sampling"
- Flag a multi-turn harness that drops `reasoning_content` / `tool_calls`, or trims dynamically loaded tool definitions out of history. — kimi.md rule #3a, § K3.1 Pro
- Flag effort changes mid-session: they invalidate the prefix cache, so the level should be chosen before the conversation starts. — kimi.md rule #3
- Keep functional persona, few-shot and numbered steps. Restore them if a GPT-tuned port stripped them. — kimi.md rules #1, #2, #5
- Don't rely on the built-in web search for fresh facts; supply reference text. Don't cite K3.1 Pro benchmark numbers from the quickstart. — kimi.md § K3.1 Pro

### Kimi K2.6 / K2.7-Code (retiring)

- `[CRITICAL]` Flag the retirement: closed to new users 31.10.2026, retired by end of December 2026. Migrate to K3.1 Pro by removing `thinking` and using `reasoning_effort`. — kimi.md § K2.7-Code, § K2.6 "Migration off K2.6"
- If swarm work is expected but not framed, add the explicit decomposition line. — kimi.md § K2.6 Agent Swarms protocol
- On K2.7-Code, strip "don't overthink" and reasoning-shaping cost controls; model choice is the only lever. — kimi.md § K2.7-Code
- Prompts still pinned to `kimi-k2.5` / `kimi-k2-*` ids get 404. This is a migration case. — kimi.md § K2.6, `archive/models/kimi.md`

### GLM-5.3 / 5.3-Flash / FlashX (or migrated from GLM-5.2)

- `[CRITICAL]` A config sending `thinking.type: "disabled"` fails. Per Z.ai's notice, switch to `enabled` + `reasoning_effort: low` before changing the model id, and delete the matching "don't think" prose. — glm.md § GLM-5.3 Migration
- Under a heavy host prompt (Claude Code Router etc.), set `reasoning_effort` out-of-band first. Keep the `<reasoning_content>` re-injection only for routers that don't forward the param. — glm.md rule #1
- Keep the load-bearing `AGENTS.md` under ~4 KiB; the 1M window does not lift that limit. — glm.md rule #4, § router pattern
- Strip identity pinning and self-identification checks; use functional roles only. — glm.md rule #3
- For long-horizon tasks, use the vendor's shape: analyse, build, run/compare, iterate, then report what was verified and what risks remain. Add "don't fabricate data; cite sources" where business data is involved. — glm.md rule #7
- Check the endpoint and the `clear_thinking` setting before diagnosing multi-turn "forgetting". The API and the Coding Plan default Preserved Thinking differently. — glm.md § GLM-5.3 Migration, § FlashX

### GLM-5.2

- `reasoning_effort` (`high`/`max`, default `max`) is the primary thinking lever. `none`/`minimal` turn thinking off, so a "no thinking" config is valid here (unlike on 5.3). — glm.md § GLM-5.2
- Same rules #1, #3, #4 as above. Flag that 5.3 is the forward path and that its migration breaks `disabled`. — glm.md § GLM-5.3 Migration

### Qwen3.8-Max / Qwen3.8-Flash (or migrated from Qwen3.7-Max)

- Establish the surface first: hosted Max, hosted Flash, open 2.4T or open Flash-Next. The off-switch, the `preserve_thinking` default and the context window all differ. — qwen-frontier.md rule #3 table, § Qwen3.8-Flash
- `[CRITICAL]` On open `Qwen3.8-2.4T-A95B`, strip "don't think" lines; they are unimplementable there. On the other surfaces, replace them with `enable_thinking: false` or effort `low`, out-of-band. — qwen-frontier.md rule #3
- Flag a hosted-Flash multi-turn agent that assumes reasoning history persists: `preserve_thinking` defaults to `false` on hosted `qwen3.8-flash`. On `qwen3.8-max`, reasoning must go back in `reasoning_content`, never concatenated into `content`. — qwen-frontier.md rule #3, § Qwen3.8-Flash
- Before lowering effort for agent latency, cite the vendor caution: low effort can cost more in total. — qwen-frontier.md rule #3
- Add numbered required sections and a self-verification line, and restore granular constraints (explicit values, success criteria). — qwen-frontier.md rules #1, #2, #5
- For fact-recall tasks, supply reference text (abstention trait). — qwen-frontier.md rule #6
- Prompts pinned to `qwen-turbo` / `qwq-plus` / `qvq-max` are retired (10.10). Prompts on `qwen3.5-plus*` retire 19.01.2027; the replacement is `qwen3.8-flash`. — qwen-frontier.md header

### Qwen3.8-Omni-Flash

- For audio/video, name the clip or segment the question is about. Wording deltas otherwise `?`. Latency is fixed with `reasoning_effort`, not prose. — qwen-frontier.md § Qwen3.8-Omni-Flash

## DeepSeek, Tencent, MiniMax, Xiaomi, StepFun

### DeepSeek V4.1 Flash (or migrated from V4-Flash / Vision-Exp)

- `[CRITICAL]` If the harness pins `deepseek-v4-flash` or `deepseek-v4-flash-vision-exp`, it is already running V4.1-Flash-1017. Re-test; don't tune by id (deepseek.md § Version lineage, § V4.1 Flash).
- Global brevity lines ("be brief", "no preamble") written against V4-Flash: flag as possible overcorrection. 1017 has "shorter outputs" by design. Re-state length as a task requirement and re-measure (§ V4.1 Flash › Wording behaviors).
- Effort values: `medium` / `xhigh` silently become `high`, and `minimal` → low, `ultra` → max. An integer on the hosted API is undocumented. On open weights, 50 / 75 / 100 = low / high / max (rule #5).
- Strip "think step by step" and "don't think" prose. Off is `thinking.type: disabled` or Responses `effort: none` (rule #5).
- Don't open with "move the system bulk to the user turn" on a working agent body. The rule is practitioner-sourced, and the two non-reproductions were on archived 0731. Measure first (rule #1).
- Prompts that say "build on your earlier reasoning" in tool-less multi-turn: that reasoning is dropped, so restate conclusions visibly (rule #3).
- Images: keep them in user turns. A V4.1-specific placement restriction is `?` (§ V4.1 Flash › Headline facts).

### DeepSeek V4-Pro (0813)

- No image input on `deepseek-v4-pro`. Route multimodal work to `deepseek-flash` (§ V4-Pro).
- Demand JSON in prose even with `response_format` set (rule #6). Use XML `<context relevance>` structure for multi-section input (rule #2).
- With tools, `reasoning_content` must be round-tripped; flag "400 on second turn" as client-side (rule #3).
- Keep stable front matter byte-identical so caching pays (cache hit ~1/30 of a miss) (rule #8).
- Sampling in thinking mode is ignored or fixed. Don't design around `top_p` / `temperature` (§ V4-Pro › Headline facts).

### Tencent Hy4 preview

- Strip self-review rituals ("double-check your work"). The vendor names over-verification as a known issue (tencent.md § Hy4 › Wording behaviors).
- Reasoning is binary, `high` / `no_think`. Remove prose that tries to set an intermediate depth. Hosted knob is `?`, so ask what the endpoint passes (§ Hy4 › Headline facts).
- `[CRITICAL]` No behavioral delta beyond the documented facts. Mark persona / few-shot / literalism / emphasis `?` (§ Family-wide notes #2).
- "No system prompt" deployments still get a synthesized system message carrying the reasoning token (§ Hy4 › Headline facts).

### MiniMax M3.1-Flash (or migrated from M2.5 / M2.7)

- `[CRITICAL]` Any thinking-off configuration returns 400. Lower effort instead (other-frontier.md § MiniMax).
- Hosted default effort is `max`. Set effort explicitly per workload (§ MiniMax › Headline facts).
- Self-hosted: always send a system prompt, or the template injects a MiniMax identity line and a January 2026 cutoff. Pass the date via `current_date` (§ MiniMax › What the chat template injects).
- Remove "think before answering / reason about tool results" lines; the template appends them every turn (same §).
- M2.x via M Plan / Token Plan has already run on M3.1 Flash since 19.10. Re-test (§ MiniMax M2.5 / M2.7).

### Xiaomi MiMo-V2.6 Pro / Flash (or migrated from V2.5)

- V2.5 names error since 21.10; map V2.5-Pro → V2.6-Pro and V2.5 → V2.6-Flash (other-frontier.md § MiMo).
- Depth is on/off only (`thinking.type` / `enable_thinking`). No prose depth steering (§ MiMo › Wording consequences).
- With tools, round-trip `reasoning_content`: a 400 otherwise, and the vendor warns of degraded instruction following if history is missing (§ MiMo › Headline facts).
- Looping tool calls: check whether MOPD weights serve the id before adding anti-repetition prose. Its effect is `?` (§ MiMo › Wording consequences).

### StepFun Step 5 Preview

- Set effort explicitly. `medium` is recommended, benchmarks ran at `high`, and the omitted default is `?` (other-frontier.md § Step 5).
- Strip "don't think" lines. The model always thinks (§ Step 5 › Headline facts).
- The harness must read reasoning from `reasoning`, not `reasoning_content` (§ Shared notes #2).
- `[CRITICAL]` No model-specific behavioral claims. StepFun's prompting page is generic (§ Step 5 › Wording consequences).

---

## xAI, Mistral, Meta — Grok, Mistral Large, Muse

### xAI Grok 4.7 (or migrated from 4.6 / 4.3)

- `[CRITICAL]` if the architecture assumes a 1M window (a 4.3-era design): 4.5 / 4.6 / 4.7 have 500K. Fix: chunk, retrieve or summarize the carried context, not rewording (`grok.md` rule #5, § Grok 4.6 › The migration trap).
- Flag accumulated persistent context that can cross the 200K price step, where the whole request is billed at the higher rate (§ Grok 4.6 › The 200K pricing step).
- Strip "don't think / answer immediately": unimplementable on 4.5+. On 4.3 the off-switch is `reasoning_effort: "none"`, not prose. Raise depth complaints as a `reasoning_effort` question, and measure before recommending `xhigh` (rule #1).
- Prose moderation rules ("no profanity / graphic violence / sexual content"): suggest the `guidelines` parameter as a conversation point. Keep the prose in cross-vendor prompts, for rules outside the three categories, or when the harness can't send the array on every request (rule #10).
- Per-turn values (current time, counters) at the top of the system prompt: move them to the latest message, because they break caching (2.2–2.9× cost) (rule #11).
- A prompt-injection defense that is only an `<untrusted>` fence line is under-defended: xAI's tests needed four defenses together (rule #11).
- Harness: replay 4.7's encrypted reasoning items unchanged. `stop` / penalty params error on reasoning models (§ Grok 4.7; rule #1). No prompting guide exists, so don't invent version deltas.

### Mistral Large 4 (or migrated from Large 3 / Medium 3.5 / the October preview)

- The system prompt should "define a clear environment and use case, including guidance on how to effectively leverage tools in agentic systems". A bare persona + rules prompt is missing this (`mistral-frontier.md` § Mistral Large 4 › Wording guidance).
- Trim the tool list to what the use case needs before adding when-to-use prose (vendor Tools line, same §).
- Strip "don't think / answer immediately": Large 4 has no off-switch. Preview `none` → `low`; depth is `reasoning_effort` (§ Mistral Large 4 › The reasoning ladder; rule #3).
- Re-check prose that compensated for an instruct-only model ("work through this step by step"): the lever is now the parameter (§ Migrating from Large 3 or Medium 3.5).
- Harness: with reasoning on, content is a chunk list. Replay the full assistant message incl. the thinking chunk, and parse both a string and a chunk list at `low` (rule #3).
- Pinned `mistral-large-2512` / `mistral-medium-3-5`: flag the 2027-04-30 retirement (§ Large 3, § Medium 3.5).

### hosted Muse Wonder 1.0 / Muse Spark 1.3 (or migrated from Spark 1.2)

- Establish whether the run is interactive. Both ask clarifying questions and check in before consequential actions. For non-interactive runs, state that no one can answer, what is pre-authorized, and the ambiguity fallback; mark suppression "test it" (`meta.md` § Muse Spark 1.3 › The behavioral shift; § Muse Wonder 1.0).
- Say when each tool applies: `tool_choice` accepts only `"auto"`, and forcing returns 400. Don't promise schema-perfect arguments, because `strict` only validates the schema (family rule #3).
- Strip "don't think": `none` → 400. Depth is `reasoning_effort` (Chat Completions) / `reasoning.effort` (Responses), minimal…max. Lookups, formatting and translation → `low` (vendor) (family rule #5).
- Strip harness-side context-trimming scaffolding: the model manages its 1M window (family rule #1).
- Harness: multi-turn agents that need reasoning carried over belong on the Responses API (Chat Completions redacts it). `max` on `muse-wonder-1.0` is not Wonder Full (family rule #5; § Muse Wonder 1.0).
- From 1.2: re-test constraint restatements and coding-concision lines. Audio input → 1.2 (§ Muse Spark 1.3; rule #4).

### Muse Spark Open (self-hosted)

- `[CRITICAL]`-class: any "Reasoning effort/strength:" text in the system prompt is a live control. The template rewrites it and drops its own numeric line, so a ported `Reasoning effort: high` silently overrides the request parameter. Delete it, or pin a number deliberately (`Reasoning strength: 256.`) (`meta.md` § Muse Spark Open › `Reasoning strength: N.`).
- Must-hold rules go first: the vendor says attention favors the start of the system prompt. Buried invariants are a placement finding (§ Rules first).
- Strip prose that re-describes the `tools` array and keep only "Use [tool] when [condition]" (§ Don't restate the tools).
- Open with a specific role (the template already prepends "You are a helpful AI assistant."). Don't hand-write the date (§ Don't restate the tools; the template's system lines).
- Strip "don't think": reasoning can't be turned off. Default `xhigh`, so lower it explicitly for short tasks (family rule #5).
- Media parts go where the text refers to them (§ Tools, media, output).

### Muse Glimmer (local)

- Replace a generic "helpful assistant" opener with a specific role, and state format / length / tone constraints upfront (`meta.md` § Muse Glimmer › Wording rules).
- Strip meta-instructions about how the model works internally (same §).
- Strip parallel-call / batching instructions: one tool call per turn (same §).
- Keep (or add) pruning / summarizing of old turns in long loops. Hosted Muse's self-managed context is not documented for Glimmer (family rule #1).
- Depth is the `reasoning_strength` template argument, never prose. Use `apply_chat_template`, because hand-built prompts degrade (same §).

## Class 2 — small local

### Generic Class 2 task prompt (any family)
- Confirm the target is Class 2: dense 2–9B, or MoE ≤ ~9B active and ≤ ~35B total. Above that, use the frontier path (`matrix-small.md` header).
- `[CRITICAL]` A decision model (Clef, Decider, d1, Granite Switch `decision`) is out of scope. Say so and stop (`small-local.md` top).
- Before rewriting for thinking-off, check the model has a working switch. If it doesn't, the fix is a token budget plus strip, not prose (`antipatterns-small.md § 2`).
- Tool few-shot examples must be in the model's native call format (`antipatterns-small.md § 4`).
- A loop or repetition on llama.cpp: check the sampler first (`antipatterns-small.md § 11`).

### MiniCPM5-2B

- `[CRITICAL]` `enable_thinking` must be passed as `False` explicitly for compile-time tasks. Absent means the model decides (`small-local.md § OpenBMB MiniCPM5` 1).
- `[CRITICAL]` llama.cpp with default `min_p=0.05` → loops. Set `min_p=0.0` before touching the prompt (§ MiniCPM5 2).
- Flag a missing `--jinja` on `llama-server` (§ MiniCPM5 3).
- Tool examples in XML `<function name=…><param name=…>` with CDATA, not OpenAI JSON. Use `<tool_def_sep>` to place the tool block deliberately (§ MiniCPM5 4).
- Behavioral axes `?`: don't import Gemma/Qwen numbers (`matrix-small.md` reading guide).

### Nemotron-Labs Dory 4B

- Strip "think less / answer directly" prose. Depth is `num_loops` (serving) and reasoning is always on in vLLM (`small-local.md § NVIDIA Nemotron-Labs Dory`).
- Budget `max_tokens` for the reasoning trace and strip it (`antipatterns-small.md § 2`).
- `[CRITICAL]` for coding agents: don't rely on `force_nonempty_content`, which is absent from the shipped template. Check for empty `content` in the harness (§ Dory).
- Don't duplicate the template's `<IMPORTANT>` tool-format reminder in the system prompt (§ Dory, tool format).
- Keep the system prompt English (EN-only model). Flag non-EN input as untested (§ Dory).
- Sampling 1.0 / 0.95, not the project's 0.1 default (`antipatterns-small.md § 11`).

### LFM3-1.7B (or its Omni / VL siblings)

- Pass `enable_thinking=False` and also keep the strip. The vendor says the switch "can be inconsistent" (`small-local.md § LFM3-1.7B`).
- Send your own system prompt (the card's format example shows the default Liquid system line) (§ LFM3-1.7B).
- Deferred tools: use the vendor's verbatim `search_tools` system line (§ LFM3-1.7B).
- Flag knowledge-heavy or agentic-coding use. The vendor advises retrieval and advises against agentic coding (§ LFM3-1.7B).
- Omni: output modality follows the system prompt. Add `Respond with text.` (or the ASR line) at the end of the system prompt (§ siblings).

### LFM2.5-24B-A2B

- `[CRITICAL]` No thinking-off exists. Remove "don't think" lines and size `max_tokens` for the trace (`small-local.md § LFM2.5-24B-A2B`; `antipatterns-small.md § 2`).
- If the parser expects JSON tool calls, say so in the system prompt (the vendor-documented switch). Otherwise expect Pythonic calls (§ LFM2.5).
- Inside OpenClaw / Hermes Agent / Pi / OpenCode, compare against the harness's stock system prompt before rewriting (§ LFM2.5, harness-native).
- Flag knowledge-heavy tasks without retrieval (§ LFM2.5).

### Ministral 3 3B / 8B (migrated from 2024 Ministral 3B / 8B)

- The 2410 API ids are retired. Confirm the build (`small-local.md § Mistral / Ministral`).
- Measured Ministral 3B cells (markdown fences, 2-pass +10pp, self-verify −40pp) are a proxy on 2512. Flag them as such (`matrix-small.md` reading guide).
- The system prompt should define the environment and use case; keep tools minimal (vendor card, § Mistral / Ministral).
- Temperature below 0.1 (vendor); this fits `antipatterns-small.md § 11`.

### IBM Granite 4.2 3B / 8B

- Pick the thinking mode in the template or serving call, not in prose (`small-local.md § IBM Granite`).
- Don't depend on earlier-turn CoT (stripped by default).
- Few-shot tool examples on the 3B; agentic loops viable on the 8B, scoped tightly.
- If it is actually **Granite Switch 4.1**: chat comes from the 4.1 base, so the 4.2 mode notes don't apply. The adapter is picked via `adapter_name`, and `decision` is out of scope (§ Granite Switch paragraph).

