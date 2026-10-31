# Archive — legacy sections from `models/deepseek.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

## DeepSeek V4-Flash (April 24, 2026 — official release 2026-07-31; id now routed to V4.1 Flash)

### Headline facts

- **Architecture:** 284 B total parameters, 13 B activated per token
- Same 1 M context, same DSA, same family character as V4-Pro — at a much lower cost / latency profile
- **Official release 2026-07-31** (`deepseek-v4-flash` name unchanged; the `-0731` build is the same architecture as the Preview, re-post-trained) — Terminal-Bench 2.1 82.7, DeepSWE 54.4, CyberGym 76.7, Toolathlon 70.3 (DeepSeek's own harness figures)
- **Self-host viable** at a much smaller hardware footprint than V4-Pro

### Wording adjustments from V4-Pro → V4-Flash

- **More granular spec needed** — Flash is less forgiving of vague constraints
- **Stronger user-prompt priority** — the smaller model leans harder on the user instruction
- **Tighter context budgets** — even though nominal context is 1 M, Flash reasoning degrades earlier than Pro

### When V4-Flash specific tuning helps

- Cost-sensitive production agents
- High-volume batch tasks
- Latency-critical interactive UIs where V4-Pro's reasoning depth isn't needed

## DeepSeek V3.2 / V3.2-Speciale (December 1, 2025 — legacy)

### Headline facts

- **Released:** 2025-12-01 per the official changelog (date corrected 2026-10-31; earlier text said February 15, 2026). V3.2-Speciale ran on a temporary endpoint until 2025-12-15
- **V3.2-Speciale:** high-compute variant
- **Architecture predates DSA** — historical token-cache economics apply (no $0.028/M caching)
- **Multi-turn API:** `reasoning_content` was **rejected** on input — the reverse of V4

### When V3.2 specific tuning matters

- Production stacks still on V3.2 that haven't migrated to V4
- Cost-sensitive deployments where V3.2 pricing wins (V4 may be cheaper now, but historic deployment lock-in matters)

### Migration V3.2 → V4

Primary breaking changes:
- **API:** `reasoning_content` round-trip behavior inverts. Multi-turn flows that worked on V3 silently break on V4 (or vice versa).
- **Prefix cache hashing:** differs between V3.2 and V4 — caches don't transfer.
- **Tokenizer specifics:** if you hard-coded V3.2 tokenizer assumptions, audit before the port.

Wording-side, the rules don't change much (user-prompt priority, XML context structure, JSON-in-prose all worked on V3.2 and continue to work on V4). The migration cost is API-level, not prompt-level.

## DeepSeek R1 (lineage note)

R1 was the reasoning-first variant in the V3 era. Key historical facts that affect prompt-atlas reviews when the user mentions R1:

- **Tool calls and reasoning were mutually exclusive** — V4 unified them
- **`reasoning_content` was rejected** on multi-turn input (same as V3.2)
- **R1-era prompts** that worked around the tool/reason exclusion can be simplified on V4

If you encounter a prompt that explicitly says "first reason, then in a follow-up call use the tool" and the target is V4, flag it as obsolete and simplify.

## DeepSeek V4-Flash-Vision-Exp (August 21, 2026 — experimental vision sibling; id now routed to V4.1 Flash)

The family's first multimodal model — the answer to "V4 is text-only". Released 2026-08-21 per the official changelog; accessed with `model='deepseek-v4-flash-vision-exp'`. It is the V4-Flash text model with image / screenshot understanding added; **on pure-text benchmarks it is on par with V4-Flash, and on agent benchmarks requiring visual understanding it lands close to Claude Opus 4.8** (official claim; third-party coverage: beats Opus 4.8 on 3 of 11 agent benchmarks).

### Headline facts

- **API id:** `deepseek-v4-flash-vision-exp` (experimental build — versioned like `-exp` models, no stability promise)
- **Text capability:** on par with official V4-Flash (agent, reasoning, world knowledge)
- **Vision-agent benchmarks** (DeepSeek Harness minimal mode, max effort, topp 0.95, temperature 1.0): Terminal Bench 2.1 83.9, NL2Repo 57.7, DeepSWE 59.3, DSBench-Hard 63.6, ApexBench (Pass@1) 36.5, Agents' Last Exam 27.3, Chartography 64.3, ZeroBench (Pass@5) 35.0
- **Formats:** JPEG, PNG, GIF, WebP (detected from file content, not name/MIME); max 600 images per request; auto-resize → up to **384 tokens per image** (2000×2000 and 5000×5000 bill the same); max 8192 px per side (drops to 4096 px with 15+ images)
- **Transport:** standard OpenAI-compatible Chat Completions (base64 / URL / Files API `file_id`), Anthropic-compatible `/messages` surface, and OpenAI Responses API (`input_image` parts)
- **`detail` field** on image inputs: `low` (downscaled 512×512), `high`/`original` (kept), `auto` (currently = original) — a tuning lever for cost vs fidelity

### Wording behaviors that matter

1. **Images live in user messages only — structurally reinforced rule #1.** Images in `system` or `assistant` messages return a 400 error; only `deepseek-v4-flash-vision-exp` accepts images at all (other models 400 with "This model does not support image"). This is the family's user-prompt-priority made *structural* for multimodal content: the visual context *cannot* ride in the system turn. A prompt review on this model should never place task-critical visual data in system position — move it to user blocks even where the system body otherwise stays.
2. **Same thinking regime as V4-Flash** — effort model (`thinking` toggle + `reasoning_effort` low/high/max, rule #5 applies); benchmarks above were run at `max`. Reasoning is not a prose lever here either.
3. **Prompt the text around images explicitly** — the model does not invent which image deserves attention; name the image or the order ("in the first screenshot…", "comparing chart 2 and 3…") when the task is selective.
4. **Family rules #2 (XML context structure) and #6 (JSON-in-prose) carry over unchanged** for mixed text+image tasks.

### When V4-Flash-Vision-Exp specific tuning helps

- Screenshot-driven agent loop (UI automation, visual QA, chart extraction) where DeepSeek economics beat the closed vision flagships
- Cost-sensitive multimodal pipelines — text parity with V4-Flash means the vision surcharge buys only the encoder

### When NOT to invest

- Long-horizon closed-frontier agentic work (same gap as V4-Flash, see V4-Pro section)
- Tasks where the vision benchmark gap to Opus-class multimodal models matters (official wording: "close to" Opus 4.8, not ahead)
