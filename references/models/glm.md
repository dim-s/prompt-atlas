# Model-specific wording differences — Z.ai GLM family

What changes about how you should PHRASE prompts for GLM-5.2, GLM-5.1, GLM-5, and GLM-4.6. Companion to `claude.md`, `gpt.md`, `gemini.md`. This reference stays focused on wording — not API parameters, infrastructure, or pricing.

GLM lives in a different agentic ecosystem than Claude / GPT / Gemini: it's most commonly accessed **through cross-tool routers** (Claude Code Router, OpenCode, Cline, Kilo Code, Cursor, OpenClaw) rather than a vendor-native CLI. That fact dominates how prompts must be shaped — see § *The Claude-Code-router pattern* below.

---

## Family-wide rules (apply to all current GLM versions)

These hold across GLM-4.6 → GLM-5 → GLM-5.1. Version-specific notes follow.

### 1. Thinking is enabled by default — but heavy system prompts suppress it

Z.ai's `/chat/completions` endpoint has reasoning enabled by default. The model itself decides whether to engage chain-of-thought through internal judgment.

**The catch:** Claude Code's extensive system prompt — and other heavy host system prompts (OpenCode, Cline, Kilo Code) — measurably **suppress GLM's reasoning judgment**, causing the model to rarely engage thinking even though the API permits it.

This is unique to GLM among current frontier vendors. Claude, GPT-5.x, Gemini 3, and Kimi K2.6 do not exhibit a comparable "thinking gate" that a host prompt can close.

**Mitigation pattern (used by Claude Code Router and similar projects):** rather than stripping the host's system prompt, **inject explicit reasoning directives** that re-open the gate:

```
Before answering, write detailed reasoning in <reasoning_content>...</reasoning_content> tags. Maintain chain-of-thought even for seemingly simple problems. Structure responses as: reasoning block first, conclusion second.
```

Notes on the markers:
- `<reasoning_content>` is preferred over `<thinking>` — standard thinking tags don't reliably trigger reasoning on GLM (likely a training-data artifact)
- The custom markers also keep the reasoning visible in the conversation flow rather than being filtered into a separate field

If you're writing a prompt targeting GLM specifically **without** a host system prompt (direct API usage), you can rely on the endpoint default — but flag the assumption so future readers know why the prompt has no explicit reasoning instruction.

**GLM-5.2 update (June 2026):** GLM-5.2 exposes an explicit `reasoning_effort` parameter (`"high"` / `"max"`) alongside the `thinking` toggle — reasoning depth is now a runtime knob like every other frontier vendor. This demotes the `<reasoning_content>` prose re-injection to a **fallback** for routers that don't forward `reasoning_effort` to Z.ai's endpoint. Where the parameter reaches the endpoint, set `reasoning_effort` instead of injecting prose markers. The heavy-host-prompt suppression concern above is not documented as fixed, so keep re-injection in your toolkit — but reach for the parameter first. See § *GLM-5.2*.

### 2. Outcome-first, but explicit step naming tolerated

GLM is less literal than DeepSeek V4 and less outcome-strict than GPT-5.5. Process-step phrasing ("first read X, then identify Y, then propose Z") is tolerated without measurable degradation. Outcome-first remains better wording, but you don't need to aggressively strip steps when porting from a Claude prompt.

### 3. Distillation-identity confusion — avoid hard identity pinning

GLM-5 and GLM-5.1 have a documented quirk: when asked who created them, the model occasionally responds *"I am Claude, created by Anthropic."* This is a training-data artifact (likely from distillation) and is not stable across queries.

Prompt-wording implication:

- **Don't write prompts that test or enforce identity** ("You are GLM-5.1 by Z.ai" + downstream "Verify you are GLM-5.1 before continuing"). The verification will fail randomly.
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

GLM follows the OpenAI `tools` array convention with `parallel_tool_calls` support. No GLM-specific oddities reported in tool-calling formats. The unique behavior is that **GLM-5.1 demonstrates very long autonomous runs** — 6,000+ tool calls in a single Vector-DB-Bench optimization task per Z.ai's release post — which means tool descriptions need to be robust to many repeated invocations.

### 7. Long-horizon autonomy as a stated feature

GLM-5.1 is positioned as an "8-hour autonomous run" model. When prompting for long-horizon work, prefer:
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

## GLM-5.2 (Z.ai, June 16, 2026 — current frontier)

### Headline facts

- **Architecture:** ~753 B-parameter MoE (~40 B active), MIT license, open weights on Hugging Face. Adds an upgraded **Multi-Token Prediction (MTP)** layer for speculative decoding (~20% longer accepted token length at inference — a latency/throughput fact, not a prompting one).
- **Context:** **1M-token, "solid lossless" long context.** Z.ai claims more stable ultra-long-context performance than 5.1, "even surpassing Opus in select real-world benchmarks," after months of specialized training for long-horizon Coding Agent scenarios.
- **Benchmarks:** Terminal-Bench 2.1 **81.0** (was 62.0 on 5.1), SWE-bench Pro **62.1** (was 58.4; beats GPT-5.5's 58.6), MCP-Atlas tool-use **77.0** (> GPT-5.5's 75.3); FrontierSWE trailing Opus 4.8 by ~1%. Strongest open-weight model at release.
- **Positioning vs 5.1:** "stronger project-level context capacity, more stable long-horizon task execution, more reliable adherence to production-grade engineering standards."
- **Pricing:** ~$4.40 / M output tokens (third-party listings; the official prompting guide doesn't state it).

### The big wording change: explicit `reasoning_effort` replaces the endpoint-only lever

This is the delta that matters most for prompt-atlas. On GLM-5.1 the only practical reasoning lever under a heavy host prompt was **prose re-injection** (custom `<reasoning_content>` markers — family rule #1). GLM-5.2 exposes reasoning depth as an explicit runtime parameter, like every other frontier vendor:

- `"thinking": {"type": "enabled"}` — enabled by default; disable with `{"type": "disabled"}`.
- `"reasoning_effort": "high" | "max"` — the selectable Thinking Mode toggle.

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

### Migration from GLM-5.1 → GLM-5.2

- Prompts run forward-compatibly; the main action is to **move reasoning control from prose re-injection to the `reasoning_effort` parameter** where the router exposes it.
- Re-run 5.1 prompts tuned around thinking-suppression — you may be able to trim `<reasoning_content>` scaffolding.
- Re-validate long-context prompts: 1M lossless means patterns split for 5.1's tighter window may now fit in one pass.

---

## GLM-5.1 (Z.ai, April 7, 2026 — previous frontier)

### Headline facts

- **Architecture:** ~754 B-parameter MoE, MIT license, open weights on Hugging Face (`zai-org/GLM-5.1`)
- **Trained entirely on Huawei Ascend 910B chips** — no NVIDIA. Production-side fact, doesn't affect prompting.
- **Benchmarks:** SWE-Bench Pro 58.4 (#1 globally at release, ahead of GPT-5.4 at 57.7 and Claude Opus 4.6 at 57.3)
- **Long-horizon autonomy:** 8-hour self-review loops demonstrated; thousands of iterations
- **Available through:** Z.ai chat (chat.z.ai), official API, Claude Code (via router), Cursor, Cline, Kilo Code, OpenCode, OpenClaw — same model, multiple host prompts
- **Self-hosting:** vLLM / SGLang / xLLM with `tensor-parallel-size 8`, requires ~1.49 TB storage unquantized

### Wording behaviors that matter

- **Long-horizon framing:** ask for verification loops and stop conditions, not turn-by-turn steps
- **Outcome-first beats step prescription** (matches Claude/GPT direction), but step prescription is tolerated
- **Identity-pinning fails** (see family rule #3); use functional roles
- **Heavy host system prompts suppress thinking** (see family rule #1)

### When GLM-5.1 specific tuning helps

- Long-running autonomous coding sessions where the 8-hour endurance is the actual feature
- Cost-sensitive deployments where you'd otherwise pay frontier prices (MIT license = self-host viable)
- Workloads where SWE-Bench Pro performance is the proxy you trust

### When NOT to invest in GLM-5.1 specific tuning

- Tasks heavily dependent on broad factual recall — GLM is closer to a reasoning model than a knowledge base
- Latency-sensitive interactive UIs — the model is tuned for long-horizon, not snappy turn-taking
- Cultural-nuance work — early reviews flag this as a weak point

### Migration from GLM-5 → GLM-5.1

Minor patches. Mostly unchanged. Re-run SWE-Bench-tuned prompts if you have them — 5.1 may match performance on simpler outputs and let you trim verbose scaffolding.

---

## GLM-5 (Z.ai, February 11, 2026)

### Headline facts

- **Architecture:** 744 B-parameter MoE, 40 B active per inference
- **Focus:** system engineering (deliberate shift from GLM-4.6's pure-coding focus)
- **Sparse attention** for token efficiency
- **Two operational modes:** Chat Mode and Agent Mode (the Agent Mode produces `.docx` / `.pdf` / `.xlsx` deliverables and runs multi-step workflows)

### Wording differences from GLM-4.6

- Prompts written for GLM-4.6's coding focus may underutilize GLM-5's system-engineering bias. If your task is broader than "write this function" (e.g., "design and implement this service end-to-end"), the prompt can be more outcome-stated and less imperative.

### When GLM-5 specific tuning helps

- Workflows where you want the model to plan a multi-step engineering deliverable rather than execute a coding task
- Production setups that already targeted GLM-5 and haven't migrated to 5.1 yet

---

## GLM-4.6 (Zhipu AI, September 2025 — legacy but still in production)

### Headline facts

- **Architecture:** 355 B-parameter MoE, 32 B active, 200 K context
- **First version with thinking mode enabled by default at the API** — the family rule about thinking-gate suppression was first surfaced here
- **Tool calling supports parallel calls** and is the foundation for the Agent Swarm-style patterns formalized in GLM-5+

### Wording differences from 5.x

- Context window is half of GLM-5's (200 K vs ~400 pages-of-text equivalent) — long-context prompts written for 5.x may not fit
- The "system engineering" framing is absent — keep prompts coding-task-scoped
- Sparse attention is not as advanced as GLM-5's — long-context retrieval is weaker

### When to keep using a GLM-4.6 prompt

- Production stacks already on 4.6 with measured behavior — don't rewrite unless migrating
- Cost-sensitive cases where 4.6's pricing wins
- Self-hosted deployments with smaller hardware budgets

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
- **Reasoning lever is out-of-band, not prose** — GLM-5.2 has an explicit `reasoning_effort` param (`high`/`max`); earlier versions rely on the endpoint default. Either way, don't write "think step by step"
- **Outcome-first beats step prescription**, but not as aggressively as GPT-5.5
- **`json_schema` over `json_object`** for structured output
- **Long-horizon framing is GLM's wheelhouse** — write success criteria and stop conditions, not turn-by-turn steps
- **Router-mediated access is the norm, not the exception** — budget for host-prompt overhead
- **MIT license + open weights** mean prompts may run against self-hosted endpoints with different host prompts than the cloud — write prompts that work in both contexts

---

## Cross-vendor rules (when GLM is one of several targets)

If a prompt must run on GLM **and** Claude / GPT / Gemini:

- The 4 KiB AGENTS.md ceiling for GLM is **stricter** than Codex's 32 KiB and Claude Code's "soft 300-line" ceiling. **Strictest constraint wins** → write to GLM's budget.
- Identity pinning is fine on Claude / GPT / Gemini but breaks GLM. **Strip identity pinning.**
- `json_schema` over `json_object` works on Claude / GPT / Gemini / GLM alike — safe default.
- Heavy system prompts hurt GLM specifically. On GLM-5.2, the first mitigation is the `reasoning_effort` parameter (out-of-band, costs nothing on other vendors). If the router doesn't forward it, or you're on GLM-5.1 and earlier, fall back to injecting explicit `<reasoning_content>` directives at the top of your `AGENTS.md` as the family-wide mitigation.

When in doubt: tune for GLM, then check the result reads well on the other vendors. The reverse usually fails.
