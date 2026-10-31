# Model-specific wording differences — Z.ai GLM family

What changes about how you should PHRASE prompts for GLM-5.3, GLM-5.3-Flash / FlashX, and GLM-5.2. GLM-5.1, GLM-5 and GLM-4.6 are still listed in Z.ai's price table (no retirement notice found), but they are two or more generations behind; their sections are in `archive/models/glm.md`. Companion to `claude.md`, `gpt.md`, `gemini.md`. This reference stays focused on wording — not API parameters, context sizes, infrastructure, or pricing; a knob is named only where a prompt line would fight it.

GLM lives in a different agentic ecosystem than Claude / GPT / Gemini: it's most commonly accessed **through cross-tool routers** (Claude Code Router, OpenCode, Cline, Kilo Code, Cursor, OpenClaw) rather than a vendor-native CLI. That fact dominates how prompts must be shaped — see § *The Claude-Code-router pattern* below.

---

## Family-wide rules (apply to all current GLM versions)

These hold across GLM-5.2 → GLM-5.3 → GLM-5.3-Flash / FlashX; rules #1, #3, #6 and #7 were first observed on the archived GLM-4.6 – 5.1 and are marked where the current evidence is thinner. Version-specific notes follow.

### 1. Thinking is enabled by default — but heavy system prompts suppress it

Z.ai's `/chat/completions` endpoint has reasoning enabled by default. The model itself decides whether to engage chain-of-thought through internal judgment.

**The catch:** Claude Code's extensive system prompt — and other heavy host system prompts (OpenCode, Cline, Kilo Code) — measurably **suppress GLM's reasoning judgment**, causing the model to rarely engage thinking even though the API permits it.

This is unique to GLM among current frontier vendors. Claude, GPT-5.x, Gemini 3, and Kimi do not exhibit a comparable "thinking gate" that a host prompt can close.

**Mitigation pattern (used by Claude Code Router and similar projects):** rather than stripping the host's system prompt, **inject explicit reasoning directives** that re-open the gate:

```
Before answering, write detailed reasoning in <reasoning_content>...</reasoning_content> tags. Maintain chain-of-thought even for seemingly simple problems. Structure responses as: reasoning block first, conclusion second.
```

Notes on the markers:
- `<reasoning_content>` is preferred over `<thinking>` — standard thinking tags don't reliably trigger reasoning on GLM (likely a training-data artifact)
- The custom markers also keep the reasoning visible in the conversation flow rather than being filtered into a separate field

If you're writing a prompt targeting GLM specifically **without** a host system prompt (direct API usage), you can rely on the endpoint default — but flag the assumption so future readers know why the prompt has no explicit reasoning instruction.

**GLM-5.2 update (June 2026):** GLM-5.2 exposes an explicit `reasoning_effort` parameter (`"high"` / `"max"`) alongside the `thinking` toggle — reasoning depth is now a runtime knob like every other frontier vendor. This demotes the `<reasoning_content>` prose re-injection to a **fallback** for routers that don't forward `reasoning_effort` to Z.ai's endpoint. Where the parameter reaches the endpoint, set `reasoning_effort` instead of injecting prose markers. The heavy-host-prompt suppression concern above is not documented as fixed, so keep re-injection in your toolkit — but reach for the parameter first. See § *GLM-5.2*.

**GLM-5.3 update (August 2026):** GLM-5.3 raises the effort lever to `low` / `high` / `max` (default `max`) and **removes the off-switch entirely** — `thinking.type: "disabled"` is no longer supported, and a request that still sends it will fail. "Don't think / answer immediately" becomes **structurally unimplementable** on GLM-5.3, the same class of antipattern as Kimi K3 / K3.1 Pro / K2.7-Code and the Qwen3.8-Max open weights `Qwen3.8-2.4T-A95B` (`antipatterns.md` #38). Existing integrations that toggle thinking off must migrate to `enabled` + `reasoning_effort: low` before naming the model (Z.ai's migration notice, quoted in § *GLM-5.3*). The `<reasoning_content>` fallback remains for routers that don't forward `reasoning_effort` (and for the archived GLM-5.1 and earlier). See § *GLM-5.3*.

**GLM-5.3-Flash / FlashX update (August–September 2026):** the Flash siblings ship the same effort surface — `reasoning_effort` `low`/`high`/`max`, **defaults to `max` if not passed** (model-card wording), and the no-off-switch rule holds (`thinking.type: "disabled"` fails). Card-specific: the chat template has a `clear_thinking` flag that **defaults to `false`** — for chat scenarios pass `clear_thinking=true`, otherwise prior reasoning persists in the template (a multi-turn cost and possible context-leak lever, not a wording concern, but worth flagging when a harness mis-behaves on multi-turn). See § *GLM-5.3-Flash*.

### 2. Outcome-first, but explicit step naming tolerated

GLM is less literal than DeepSeek (V4 practitioner reading) and less outcome-strict than GPT (5.5 guide). Process-step phrasing ("first read X, then identify Y, then propose Z") is tolerated without measurable degradation. Outcome-first remains better wording, but you don't need to aggressively strip steps when porting from a Claude prompt.

### 3. Distillation-identity confusion — avoid hard identity pinning

GLM-5 and GLM-5.1 (archived) had a documented quirk: when asked who created them, the model occasionally responded *"I am Claude, created by Anthropic."* — a training-data artifact (likely from distillation), not stable across queries. It has not been re-documented for GLM-5.2 / 5.3 (`?`); the wording rule below costs nothing and stays as insurance.

Prompt-wording implication:

- **Don't write prompts that test or enforce identity** ("You are GLM-5.3 by Z.ai" + downstream "Verify you are GLM-5.3 before continuing"). On the lineage that showed the artifact, the verification failed randomly.
- **Functional roles work fine** ("You are a code-review assistant"). The artifact only surfaces when the prompt forces the model to name itself.
- **Don't rely on the model to identify itself in user-facing copy.** If end users may ask "who made you," route that question to a fixed string in your application logic, not to the model.

### 4. System prompt brevity matters (router-mediated tooling)

When GLM runs behind a router that prepends its own system prompt (Claude Code Router etc.), every kilobyte you add to **your** system content stacks on top of the host's existing prompt. Combined size is the binding constraint on GLM's reasoning quality.

Practical guidance:
- Target **<4 KiB** for the load-bearing portion of `AGENTS.md` / `CLAUDE.md` when GLM is the routed model
- Push tool-specific guidance into tool descriptions, not the system prompt (stronger pressure here than the GPT-5.5 rule)
- Trim per-tool usage hints, per-tool when-to-call examples, etc. — let the router's host prompt do that work

### 5. JSON output: `json_schema` preferred, `json_object` legacy

Z.ai's API exposes two structured-output modes: `json_object` (older method) and `json_schema` (recommended for stricter shape control). Default to `json_schema` when the receiving stack supports it.

In the prompt body itself, you can still describe the JSON shape — GLM tolerates prose-based format constraints — but the schema parameter is the load-bearing lever.

### 6. Tool calling: standard OpenAI-compatible surface

GLM follows the OpenAI `tools` array convention with `parallel_tool_calls` support (Z.ai recommends `tool_stream: true` alongside `stream: true` for streamed tool arguments). No GLM-specific oddities reported in tool-calling formats. The line runs very long autonomous loops — GLM-5.1 (archived) showed 6,000+ tool calls in a single Vector-DB-Bench optimization task per Z.ai's release post — so tool descriptions need to be robust to many repeated invocations.

### 7. Long-horizon autonomy as a stated feature

The line has been positioned for long-horizon autonomy since GLM-5.1's "8-hour autonomous run" (archived). The current docs carry it as prompt shape: the eight official "Recommended Approach" prompts on the GLM-5.3-Flash / FlashX page all run *analyse first → build → run or render and compare against the reference → iterate → deliver*, and close with an explicit report line — e.g. *"Finally, deliver the PPTX and PDF, and explain what has been verified and what risks remain uncovered."*, and for business data *"Do not fabricate any business data; cite the source for all referenced information."* When prompting for long-horizon work, prefer:
- Outcome-defined success criteria over per-step procedure
- Explicit verification loops ("after each major change, run the test suite; if failing, return to planning")
- Clear stop conditions ("stop when all tests pass and the design doc is updated")

This style mirrors the long-horizon prompting patterns from Claude Code agent prompts — it travels well, no GLM-specific rewrite needed.

### 8. Persona blocks work, with the identity caveat

Functional persona blocks ("You are a backend engineer focused on Python web services") are fine and helpful, similar to Claude. Just don't pin the model's own vendor identity inside the persona (rule #3).

### 9. Few-shot examples: helpful, no special caveats

Unlike GPT-5.5 where few-shot examples can hurt reasoning, GLM treats few-shot examples conventionally — they help format steering and reasoning calibration. Default of 3–5 examples is reasonable.

### 10. The vendor name shifts: Zhipu AI → Z.ai

Z.ai is the rebranded name as of late 2025 / early 2026. The same company, same models. Internal references in older docs may still say "Zhipu"; the API host is `api.z.ai`. Don't burn prompt tokens on vendor naming.

---

## GLM-5.3-Flash (Z.ai, August 26, 2026 — cost / multimodal sibling; the former stealth "Ox Alpha")

First **natively multimodal** model in the GLM-5 series, and Z.ai's cheapest capable model. It launched anonymously as `stealth/ox-alpha` on OpenRouter on 2026-08-20 (free for one week, ~42T tokens consumed), was claimed by Z.ai on 08-26, and the MIT weights landed on Hugging Face the same day (`zai-org/GLM-5.3-Flash`).

### Headline facts

- **Architecture:** 320 B total / 18 B active per token, hybrid sparse+linear attention (Manifold-Constrained Hyper-Connections, mHC), trained on a 30T-token multimodal corpus — **native image+video input** (card specifies image/text; launch materials cite video)
- **Context:** 1M on the docs.z.ai model page (read 2026-10-31); **the model card's eval footnotes cap runs at 300,000 tokens** — the two numbers still coexist unreconciled, so treat full-window behaviour as unverified until measured (`sean-kim / Ox Alpha catch #3`)
- **Reasoning:** same effort model as GLM-5.3 — `reasoning_effort` `low`/`high`/`max`, **default `max`** when not passed; thinking cannot be disabled; a `clear_thinking` chat-template flag defaults to `false` (pass `true` for chat)
- **Cached input costs a fifth of uncached input** (Z.ai price list) — the agent-workload headline: re-sending a stable system prompt on every step stops being the architecture driver
- **Weights:** MIT, day-one on Hugging Face; FP8 set ≈306 GiB, **Hopper-class or newer GPUs** — self-hosting is a multi-GPU server question, not a workstation question; quantized builds via Ollama / llama.cpp / LM Studio / Jan
- **First-party API + ecosystem:** Z.ai API, ZCode / GLM Coding Plan (3× GLM-5.3 quota), OpenRouter / OpenCode keep the endpoint post-rename
- **Benchmarks** (vendor-reported): Terminal-Bench 2.1 84.3 (vs Opus 4.8's 85.0), **DeepSWE 63.4** (up from 5.2's 46.2 — the number to watch for third-party reproduction), Z.ai Code Bench 29.0 (a vendor-named benchmark — weak signal), GDPval-AA v2 highest per Z.ai; **vision is the admitted weak axis** (trails Gemini 3.7 Flash on BabyVision / MVbench)

### Wording behaviors that matter

- **Multimodal input doesn't change the wording rules.** Family rules #1–#4 hold (heavy host prompts suppress the thinking gate — the <4 KiB AGENTS.md ceiling survives; `reasoning_effort` is the primary mitigation, `<reasoning_content>` re-injection the fallback). Don't write vision-specific prose that the model can't act on; image content blocks ride in the message structure, not in wording.
- **The "don't think" request is a migration error, same as GLM-5.3** — no mode without reasoning; depth is `low`/`high`/`max` only.
- **Cached-input economics reward stable front matter** — like DeepSeek's cache-hit rule (`deepseek.md` rule #8), a stable system block is now cheap to re-send; prefer stability over terseness in the persistent part.
- **Verbosity / latency are real tradeoffs** (trade coverage: tuned for agentic task completion, spends tokens thinking) — for latency-critical UIs measure `reasoning_effort: low` before rewriting the prompt.
- **Long-context sizing:** with 1M unverified, size retrieval/reasoning pipelines against the number you can verify (300K), then test upward — do not architect on the advertised figure.
- Identity-pinning still fails (family rule #3) — functional roles only.

### Migration from GLM-5.2 → GLM-5.3-Flash

- Same parameter migration as GLM-5.3: anything sending `thinking.type: disabled` breaks — use `enabled` + effort `low`.
- Prompts tuned for 5.2 run forward-compatibly; expect more thinking tokens at default `max`.
- If the port is motivated by price, treat it as a behavior re-test: benchmarks are close to GLM-5.3 on coding, but vision-heavy or long-context-at-1M loads are not covered by vendor numbers.

### GLM-5.3-FlashX (`glm-5.3-flashx`) — the fast tier of the same model

Documented on the same docs.z.ai page as Flash (one model card, two ids): *"GLM-5.3-FlashX is now live, delivering inference speeds of **200 tokens/s**"*. No separate architecture, prompting guidance or knob set is given, so § GLM-5.3-Flash applies as written. The page's recommended settings for the pair, verbatim: *"`temperature: 1`, `top_p: 0.95`, and `reasoning_effort: max`. `thinking.type` only supports `enabled`; we recommend setting `thinking.clear_thinking: false`. For streaming requests, we recommend enabling both stream: true and `tool_stream: true`."* Two review notes:

- **`clear_thinking` points opposite ways on two surfaces.** The hosted API recommends `false` (Preserved Thinking — and then *"you must return the complete, unmodified reasoning_content"*); the open-weight card's template advice for chat is `true`. Check which surface a multi-turn harness targets before calling either a bug.
- Not on the GLM Coding Plan: *"(GLM-5.3-FlashX is not yet available on the plan)"*. Its release date is not stated on any vendor page read (a 21.09 date from secondary reporting is unverified).

**GLM-5.3-FlashX-Legal** (`glm-5.3-flashx-legal`) — a legal-workflow post-train; *"API access is available to selected enterprise customers by application."* Out of general scope; no wording guidance beyond Z.ai's advice to run it in an agent with legal MCP services.

---

## GLM-5.3 (Z.ai, mid-August 2026 — current frontier)

Released in mid-August 2026 (launch post 17.08, per coverage cited in the briefing). **Same base model as GLM-5.2 — every gain comes from post-training scaling.** Z.ai frames it as the most capable open-weights coding model yet.

### Headline facts

- **Same base as 5.2; all gains from post-training** — no architecture change; the migration is behavioral, not architectural
- **+50% on Z.ai Code Bench** (in-house benchmark) over GLM-5.2; open-weight SOTA on **Terminal Bench 3.0 (28.3** vs 4.6 on 5.2) and **Agents' Last Exam (28.5** vs 23.8)
- **Emergent cyber:** **CyberGym 84.5** (best published result; ahead of GPT-5.6 Sol 83.6) and **ExploitBench 54.4** — more than double GLM-5.2's 24.4 (gains largest further up the exploitation chain)
- **Text-only, 1M context, 128K max output**
- **Reasoning is always on and cannot be disabled**: `thinking.type` only accepts `enabled`; `reasoning_effort` `low` / `high` / **`max`** (default `max`; vendor recommends `max` for coding). Requests sending `thinking.type: "disabled"` fail — migrate to `enabled` + effort `low` first
- **Open weights:** `zai-org/GLM-5.3` on Hugging Face (repo created 25.08, licence tag `other` — not MIT like 5.2 / 5.3-Flash; per the HF API)

### Wording behaviors that matter

- Family rules #1–#4 apply (heavy host prompts still suppress thinking-gate engagement — the 1M context and new knobs do not lift the <4 KiB AGENTS.md ceiling; `reasoning_effort` is the primary mitigation, `<reasoning_content>` re-injection the fallback)
- **The "don't think" request is now a migration error, not just inert prose** — on 5.3 there is no mode without reasoning; the only knob is depth (`low`/`high`/`max`)
- Long-horizon framing remains GLM's wheelhouse — outcome-defined success criteria, verification loops, stop conditions
- Identity-pinning still fails (family rule #3) — functional roles only

### Migration from GLM-5.2 → GLM-5.3

- **Mandatory parameter migration — Z.ai's notice, verbatim:** *"Migration Notice: If your application currently uses **`thinking.type: "disabled"`**, please change it to **`enabled`** and set **`reasoning_effort`** to **`low`** before updating the model ID to **`glm-5.3`**. Otherwise, the request will fail."* That is the replacement for the old no-thinking workloads; the matching prompt-side change is to delete any "don't think / answer directly" line.
- **The two endpoints validate effort differently.** On the standard API, *"For GLM-5.3 and GLM-5.3-FLASH, only `max`, `high` and `low` are supported. Any other input will result in an error."* On the Coding Plan, *"`none`, `minimal`, and `low` are mapped to `low`"* (and `medium`→`high`, `xhigh`→`max`) — so a harness sending `none` there gets `low` thinking silently, not an off-switch.
- **Preserved Thinking defaults differ too:** *"enabled by default on the Coding Plan endpoint and disabled by default on the standard API endpoint"*; enabling it on the API (`"clear_thinking": false`) requires returning the complete, unmodified `reasoning_content`.
- Prompts tuned around thinking-suppression on 5.1/5.2 run forward-compatibly; re-test the `<reasoning_content>` scaffolding — you may be able to rely on `reasoning_effort` alone
- Budget for the default `max` effort on coding tasks — it consumes more thinking tokens than 5.2's normal mode

---

## GLM-5.2 (Z.ai, June 16, 2026 — previous frontier)

### Headline facts

- **Architecture:** ~753 B-parameter MoE (~40 B active), MIT license, open weights on Hugging Face. Adds an upgraded **Multi-Token Prediction (MTP)** layer for speculative decoding (~20% longer accepted token length at inference — a latency/throughput fact, not a prompting one).
- **Context:** **1M-token, "solid lossless" long context.** Z.ai claims more stable ultra-long-context performance than 5.1, "even surpassing Opus in select real-world benchmarks," after months of specialized training for long-horizon Coding Agent scenarios.
- **Benchmarks:** Terminal-Bench 2.1 **81.0** (was 62.0 on 5.1), SWE-bench Pro **62.1** (was 58.4; beats GPT-5.5's 58.6), MCP-Atlas tool-use **77.0** (> GPT-5.5's 75.3); FrontierSWE trailing Opus 4.8 by ~1%. Strongest open-weight model at release.
- **Positioning vs 5.1:** "stronger project-level context capacity, more stable long-horizon task execution, more reliable adherence to production-grade engineering standards."

### The big wording change: explicit `reasoning_effort` replaces the endpoint-only lever

This is the delta that matters most for prompt-atlas. On GLM-5.1 the only practical reasoning lever under a heavy host prompt was **prose re-injection** (custom `<reasoning_content>` markers — family rule #1). GLM-5.2 exposes reasoning depth as an explicit runtime parameter, like every other frontier vendor:

- `"thinking": {"type": "enabled"}` — enabled by default; disable with `{"type": "disabled"}`.
- `"reasoning_effort": "high" | "max"` — the selectable Thinking Mode toggle; `max` is the default. The current Deep Thinking page also lists `xhigh`, `medium`, `low`, `minimal`, `none` for 5.2: *"`none` or `minimal` indicate that the model stops thinking; `low`/`medium` are mapped to `high`; `xhigh` is mapped to `max`."*

Implications:

- The reasoning knob is now **out-of-band, not prose** — same rule as all other vendors. Don't write "think step by step"; set `reasoning_effort`.
- The `<reasoning_content>` re-injection workaround **drops to a fallback** — reserve it for router setups that don't forward `reasoning_effort` to Z.ai's endpoint.
- Heavy-host-prompt thinking-suppression (family rule #1) is not documented as fixed; treat `reasoning_effort` as the primary mitigation now, re-injection as the fallback.

### Official prompting patterns (from Z.ai's GLM-5.2 guide)

Z.ai documents recommended prompt shapes for coding workflows — all outcome-first and long-horizon:

- **Codebase audit:** *"Please read the current project and output a system architecture map, core module responsibilities, key API contracts, major data flows, core call chains, potential technical debt, and the engineering constraints that must be followed in future refactoring."*
- **Refactoring:** enable "/goal mode"; ask for an execution plan, impact scope, and risk boundaries *before* implementation.
- **Standards enforcement:** put standards in `CLAUDE.md` / `Agent.md` and request strict adherence with explicit verification.
- **Persona:** the official example uses a functional persona ("a senior full-stack software engineer…") — consistent with family rule #8 (functional persona OK, no identity pinning).

### Wording behaviors that matter

- **1M lossless context changes the calculus for capability, not for budget.** GLM-5.2 genuinely retains long context well — you no longer need to fear a long spec won't be *held*. But the <4 KiB load-bearing ceiling for router-mediated setups (family rule #4) still holds for a different reason: heavy host prompts stack tokens and can still suppress reasoning. Big-context capability ≠ license to bloat the system prompt.
- **Long-horizon framing is even more its wheelhouse** — write outcome-defined success criteria, verification loops, and stop conditions, not turn-by-turn steps.
- **Identity-pinning still fails** (family rule #3) — functional roles only.

### Migration from GLM-5.1 (archived) → GLM-5.2

- Prompts run forward-compatibly; the main action is to **move reasoning control from prose re-injection to the `reasoning_effort` parameter** where the router exposes it.
- Re-run 5.1 prompts tuned around thinking-suppression — you may be able to trim `<reasoning_content>` scaffolding.
- Re-validate long-context prompts: 1M lossless means patterns split for 5.1's tighter window may now fit in one pass.

---

## The Claude-Code-router pattern (cross-tool routing)

GLM is commonly accessed not through a Z.ai-native CLI but through **routers** that translate Claude Code (or Codex, or Cline) API calls to Z.ai's endpoint. The most common is [musistudio/claude-code-router](https://github.com/musistudio/claude-code-router).

The router approach has wording implications:

1. **The host system prompt (Claude Code's) is always present.** You can't strip it. Your `AGENTS.md` / `CLAUDE.md` stacks on top.
2. **The router may inject `<reasoning_content>` directives** to re-open GLM's thinking gate. Check the router's config — your prompt should be compatible with its directive style (don't write conflicting instructions).
3. **MCP tool descriptions travel through the router as-is.** Tool description quality matters as much here as on GPT-5.5.
4. **Tokens are doubled-counted:** Claude Code's system prompt + your `AGENTS.md` + router injection + your message all share the GLM context budget. Keep your contribution small.

**Practical wording rule for routed GLM:**

> Treat the prompt token budget as ~50 KiB of host-prompt overhead before your content. Write `AGENTS.md` as if the file is being read on a smaller-context model, even though GLM technically has 200K+ available.

---

## Cross-model wording principles (GLM side)

Mirror of the family-wide section, distilled into rules for cross-model reviews:

- **Heavy system prompts suppress GLM thinking** — not a Claude/GPT problem, very real on GLM
- **Functional persona OK, identity-pinning fails** (distillation artifact)
- **Reasoning lever is out-of-band, not prose** — GLM-5.3 / 5.3-Flash / FlashX have `reasoning_effort` (`low`/`high`/`max`, default `max`, no off-switch, `clear_thinking` flag — API recommends `false`, Flash card template `true` for chat); GLM-5.2 has `high`/`max` + toggle; the archived 5.1 and earlier rely on the endpoint default. Either way, don't write "think step by step"
- **Outcome-first beats step prescription**, but not as aggressively as GPT-5.5
- **`json_schema` over `json_object`** for structured output
- **Long-horizon framing is GLM's wheelhouse** — write success criteria and stop conditions, not turn-by-turn steps
- **Router-mediated access is the norm, not the exception** — budget for host-prompt overhead
- **Open weights** (MIT on GLM-5.2 and 5.3-Flash; the `zai-org/GLM-5.3` repo is tagged `license:other`) mean prompts may run against self-hosted endpoints with different host prompts than the cloud — write prompts that work in both contexts

---

## Cross-vendor rules (when GLM is one of several targets)

If a prompt must run on GLM **and** Claude / GPT / Gemini:

- The 4 KiB AGENTS.md ceiling for GLM is **stricter** than Codex's 32 KiB and Claude Code's "soft 300-line" ceiling. **Strictest constraint wins** → write to GLM's budget.
- Identity pinning is fine on Claude / GPT / Gemini but breaks GLM. **Strip identity pinning.**
- `json_schema` over `json_object` works on Claude / GPT / Gemini / GLM alike — safe default.
- Heavy system prompts hurt GLM specifically. On GLM-5.3 / 5.3-Flash / FlashX the first mitigation is `reasoning_effort` (`low`/`high`/`max`); on GLM-5.2 it's `reasoning_effort` (`high`/`max`); if the router doesn't forward the parameter, or you're on the archived GLM-5.1 and earlier, fall back to injecting explicit `<reasoning_content>` directives at the top of your `AGENTS.md` as the family-wide mitigation. And don't write "answer without reasoning" for any 5.3-family target — there is no mode without it.

When in doubt: tune for GLM, then check the result reads well on the other vendors. The reverse usually fails.

---

## Source notes

- Z.ai docs read 2026-10-31 (raw): GLM-5.3 guide [docs.z.ai/guides/llm/glm-5.3](https://docs.z.ai/guides/llm/glm-5.3.md) (migration notice, effort table); Deep Thinking [docs.z.ai/guides/capabilities/thinking](https://docs.z.ai/guides/capabilities/thinking.md) (API vs Coding Plan effort validation, 5.2 ladder); Thinking Mode [docs.z.ai/guides/capabilities/thinking-mode](https://docs.z.ai/guides/capabilities/thinking-mode.md) (Preserved Thinking defaults, `clear_thinking`); GLM-5.3-Flash/FlashX page [docs.z.ai/guides/vlm/glm-5.3-flash](https://docs.z.ai/guides/vlm/glm-5.3-flash.md) (FlashX 200 tokens/s, recommended settings, plan note, the eight Recommended Approach prompts, FlashX-Legal); migration guide [docs.z.ai/guides/overview/migrate-to-glm-new](https://docs.z.ai/guides/overview/migrate-to-glm-new.md) (`tool_stream`); price table / overview [docs.z.ai/guides/overview/pricing](https://docs.z.ai/guides/overview/pricing.md) (5.1 / 5 / 4.7 / 4.6 still listed; Flash cached-input ratio); HF API for `zai-org` (GLM-5.3 repo date)
- GLM-5.3-Flash Hugging Face model card ([huggingface.co/zai-org/GLM-5.3-Flash](https://huggingface.co/zai-org/GLM-5.3-Flash), read 2026-08-29) — 320B/18B params, MIT, mHC hybrid architecture, 30T-token multimodal corpus, `reasoning_effort` low/high/max with default `max`, `clear_thinking` chat-template flag (default `false`), 300K-context evaluation footnotes, serving stacks (SGLang/vLLM/TokenSpeed/KTransformers)
- Third-party launch analysis (Sean Kim, 2026-08-29, [blog.imseankim.com](https://blog.imseankim.com/glm-5-3-flash-ox-alpha-42-trillion-tokens-openrouter-mit-open-weights-4-catches/)) — the stealth-launch timeline, the 1M-vs-300K context discrepancy, pricing ($0.15/$0.03/$0.50), ~306 GiB FP8 weight set on Hopper+, the weak-vision axis, and the "entirely on Chinese AI chips" claim (vendor-only, unverified)
- Z.ai announcement / trade coverage (z.ai/blog/glm-5.3-flash, Bloomberg 26.08, SiliconANGLE 26.08) — Ox Alpha confirmation, coding-plan quota, terminal-bench/DeepSWE figures
