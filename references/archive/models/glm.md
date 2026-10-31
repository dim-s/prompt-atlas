# Archive — legacy sections from `models/glm.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

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
