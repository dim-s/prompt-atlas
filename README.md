# prompt-atlas

An auditable, citation-driven skill for **reviewing and improving the wording of LLM prompts** across all major current model families. Every recommendation cites a documented behavior (model × axis cell) so the user can challenge it from data, not authority.

License: [CC-BY-SA 4.0](./LICENSE). Built on the methodology of [matrix-citation prompt review](references/matrix.md).

---

## Coverage

### Class 1 — Frontier agent-facing prompts (14 vendor families)

Current lineup with ids, status and the reasoning knob per model: [`references/lineup.md`](references/lineup.md). Models two generations behind or retired by the vendor are in [`references/archive/`](references/archive/README.md).

| Vendor | Models covered |
|---|---|
| **Anthropic Claude** | **Fable 5.1 / Mythos 5.1** (Sep 2026; Mythos: Project Glasswing only) / **Opus 5.5** (Sep 22 2026) / **Sonnet 5.5** (Sep 28 2026) / **Haiku 5.5** (Oct 7 2026) / previous: Fable 5, Opus 5, Sonnet 5 |
| **OpenAI GPT** in Codex CLI | **GPT-6 Astra** (Sep 2026) / **GPT-6.1 Sol** (Sep 29 2026) / **GPT-6.1 Terra** (Oct 26 2026) / GPT-6 Sol / Luna / **GPT-Live 1** (voice, GA Sep 10) / previous: GPT-5.6 Sol / Terra / Luna |
| **Google Gemini** in Gemini CLI | **Gemini 4 Argon** (GA Oct 13 2026) / 3.1 Pro / **3.8 Flash** (also serves `gemini-3.7-flash`) / 3.6 Flash (also serves `gemini-3.5-flash`) / **3.5 Flash-Lite** |
| **Moonshot Kimi** | **K3.1 Pro** (Oct 2026) / K3 / K2.7-Code and K2.6 (closed to new users 31.10, retiring end Dec 2026) |
| **Z.ai GLM** | **GLM-5.3 / 5.3-Flash / FlashX** (Aug 2026) / GLM-5.2 |
| **Alibaba Qwen frontier** | **Qwen3.8-Max** (open weights 2.4T-A95B) / **Qwen3.8-Flash** (open `Qwen3.8-Flash-Next`) / **Qwen3.8-Omni-Flash** / Qwen3.7-Max |
| **DeepSeek** | **V4.1 Flash** (`deepseek-flash`, build 1017 since Oct 17; old Flash ids route here) / **V4-Pro** (build 0813) — full 2026 version lineage in `deepseek.md` |
| **xAI Grok** | **Grok 4.7** / **Grok 4.6** (500K context) — Grok 5 still unshipped as of 31.10 |
| **Mistral frontier** | **Mistral Large 4** (weights Oct 29 2026) / Small 4 / Ministral 14B / retiring: Large 3, Medium 3.5 (2027-04-30) |
| **Meta Muse** | **Muse Wonder 1.0** (Oct 12 2026) / **Muse Spark 1.3** / 1.2 / **Muse Spark Open** (open weights 816B-A42B, prompting guide) / **Muse Glimmer** (open 30B, prompting guide) |
| **Tencent Hunyuan** | **Hy4 preview** (Aug 2026, open weights, 1M ctx) — knob documented, behavioral axes `?` |
| **MiniMax / Xiaomi MiMo / StepFun** | **MiniMax M3.1-Flash** / **MiMo-V2.6 Pro / Flash** / **Step 5 Preview** — no vendor prompting guides; facts + `?` (`models/other-frontier.md`) |

Covers CLAUDE.md, AGENTS.md (Claude Code / Codex / Gemini variants), GEMINI.md, AGENTS.override.md, vendor-specific overrides (`AGENTS.deepseek.md`, `AGENTS.glm.md`, etc.), SKILL.md, subagent definitions, slash commands, ad-hoc prompts including headless `codex exec` / Gemini CLI invocations.

### Class 2 — Small local task-facing prompts (dense 2–9B, or MoE ≤ ~9B active / ≤ ~35B total)

- **Google Gemma** 3 4B and Gemma 4 (e2b / e4b)
- **Alibaba Qwen** 3.5 (2B / 4B / 8B / 9B) — small variants only
- **Mistral Ministral 3** 3B / 8B (2512) — the 2024 Ministral 3B / 8B API ids are retired
- **Microsoft Phi** 4-mini (3.8B) / Phi-4-mini-reasoning
- **Meta Llama** 3.2 (1B / 3B)
- **IBM Granite** 4.2 3B / 8B (switchable thinking) and Granite Switch 4.1 8B; 30B is above the boundary
- **OpenBMB MiniCPM5-2B**, **NVIDIA Nemotron-Labs Dory 4B**, **LiquidAI LFM3 / LFM2.5-24B-A2B** (Oct 2026)
- **Notable fine-tunes** — saiga (Russian Gemma), T-lite (T-Bank), Hermes (Nous tune of Llama), HORROR-Imatrix, TrevorJS uncensored variants

Covers `suites/<name>/system.md`, per-model overrides `system_<model>.md`, `prompts/<topic>/*.md`, and inline system strings for LM Studio / llama.cpp / Ollama / vLLM / MLX.

---

## What it does

Auto-triggers whenever you open, write, or review any instruction text meant to steer one of the supported model families. Produces a review with:

- **Findings** tagged `[CRITICAL] / [IMPROVE] / [POLISH]`
- **Gap analysis** tagged `[ADD]` for missing snippets that match documented failure modes
- **Concrete before/after changes** with the WHY behind each
- **Model- and vendor-specific notes** when relevant
- **Cross-vendor compromises** when an `AGENTS.md` must work across multiple frontier vendors (3-way Claude+GPT+Gemini, or 4+ vendor universal)
- **Cross-class warnings** when a Class 1 technique would regress Class 2 (or vice versa)

The workflow branches on target class at Step 2a — Class 1 follows the frontier reference path; Class 2 follows the small-local reference path. Mixed-target prompts load both.

### What it does NOT cover

API/SDK parameter values (`temperature`, `max_tokens`, `reasoning_effort`, `text.verbosity`, `thinking_level`, `extra_body.thinking`, DeepSeek's `thinking` toggle + `reasoning_effort`, `enable_thinking` value choice), Codex `config.toml`, MCP server registration, hook scripts, Python/TS code around the SDKs, LoRA / SFT training data design. Those belong to vendor docs or other specialized skills.

Also out of scope: decision models (state + typed questions in, calibrated probabilities out — no system prompt to word) and speech-to-speech voice models other than GPT-Live 1.

**Known limit:** on a case-insensitive filesystem (default macOS), a Claude Code session whose working directory is `references/models/` auto-loads `claude.md` as if it were a `CLAUDE.md` — run sessions from the repo root.

The skill **surfaces** non-wording levers (e.g., "raise `effort` before rewriting", "switch to a different quant") as conversation points when relevant — but never embeds them into the artifact under review.

---

## Methodology — matrix citation

Every finding cites a specific row × column from [references/matrix.md](references/matrix.md) (frontier) or [references/matrix-small.md](references/matrix-small.md) (small local). Example finding:

> `[IMPROVE]` Persona line "You are a senior engineer" on a cross-vendor `AGENTS.md`.
> **Why:** matrix.md Table A column "Persona / 'act as'" — Gemini 3 expects +5% boost, GPT (5.6) hurts, GLM OK but no identity-pinning. Cross-vendor compromise: use methodological anchor (techniques.md §6) rather than credential persona.

This makes recommendations auditable. The user can challenge the matrix data, not the reviewer's authority.

---

## Install (global — available in all projects)

```bash
# Option 1 — clone directly into ~/.claude/skills/
git clone https://github.com/<your-user>/prompt-atlas ~/.claude/skills/prompt-atlas

# Option 2 — clone elsewhere and symlink
git clone https://github.com/<your-user>/prompt-atlas ~/dev/prompt-atlas
mkdir -p ~/.claude/skills
ln -s ~/dev/prompt-atlas ~/.claude/skills/prompt-atlas

# Verify structure
ls ~/.claude/skills/prompt-atlas/
# Expected: SKILL.md  README.md  references/  LICENSE  ...
```

Restart your Claude Code session. The skill appears as `prompt-atlas` and auto-triggers on matching contexts.

## Install (project-only)

```bash
git clone https://github.com/<your-user>/prompt-atlas .claude/skills/prompt-atlas
```

## Install for Codex CLI / Gemini CLI

The skill content is tool-agnostic — the references and methodology apply identically when invoked from Codex CLI or Gemini CLI. Discovery is host-specific:

- **Codex CLI** — place under `.agents/skills/prompt-atlas/` (repo) or `~/.agents/skills/prompt-atlas/` (user), or reference from project `AGENTS.md`
- **Gemini CLI** — place under `.gemini/skills/prompt-atlas/` or reference from `GEMINI.md`

The matrix-citation method works the same way in either host.

---

## Verify it's loaded

In a Claude Code session, ask any of these:

**Class 1 (frontier):**
> "проверь мой CLAUDE.md" (or paste any CLAUDE.md-style text)
> "адаптируй этот промпт под GPT-6.1 Sol / Codex"
> "review my AGENTS.md for cross-vendor use"
> "tune for Gemini 4 Argon"
> "почему GLM не думает в Claude Code?"
> "адаптируй под DeepSeek V4.1 Flash"

**Class 2 (small local):**
> "адаптируй промпт под gemma-4-e2b"
> "почему этот system.md не работает на qwen 2B"
> "review my suites/<name>/system.md"

**Mixed:**
> "перенеси промпт с Opus на gemma — что поменять"

The skill should engage automatically. You can also invoke it explicitly: `use the prompt-atlas skill`.

---

## Structure

```
prompt-atlas/
├── SKILL.md                       entry point, workflow, prime directive, output format
├── README.md                      this file
├── LICENSE                        CC-BY-SA 4.0
├── CHANGELOG.md                   release history per vendor update
├── CONTRIBUTING.md                how to add a new model row / vendor file
└── references/
    ├── principles.md              universal principles — applies to BOTH classes
    │
    ├── # Class 1 — Frontier agent-facing
    ├── lineup.md                  current models: ids, status, reasoning knob, section to read
    ├── model-audits.md            per-model migration / targeting checklists
    ├── matrix.md                  frontier model × axis (14 vendors × 5 tables)
    ├── artifacts.md               per-artifact checklists — CLAUDE.md / AGENTS.md / SKILL.md /
    │                              subagent / slash-command / ad-hoc
    ├── techniques.md              frontier wording snippets — outcome-first, output contracts,
    │                              creative kernel, anti-hallucination, etc.
    ├── antipatterns.md            frontier anti-patterns with fixes
    ├── patterns.md                install-once patterns — hint + anchor, creative kernel, small-model prompt
    │
    ├── # Class 2 — Small local task-facing
    ├── matrix-small.md            small-model × axis — Gemma / small Qwen / Ministral / Phi / Llama / Granite / MiniCPM / LFM / Dory
    ├── techniques-small.md        small-model snippets — skeleton, few-shot-at-end, named anti-patterns
    ├── antipatterns-small.md      12 anti-patterns specific to small models with reproduced regressions
    │
    ├── multilingual.md            non-EN prompt handling — applies to BOTH classes
    │
    ├── models/                    vendor-specific deltas
    │   ├── claude.md              Fable 5.1 / Opus 5.5 / Sonnet 5.5 / Haiku 5.5 + Fable 5 / Opus 5 / Sonnet 5
    │   ├── gpt.md                 GPT-6 Astra / 6.1 Sol / 6.1 Terra / 6 Sol / Luna / GPT-Live 1 + GPT-5.6
    │   ├── gemini.md              Gemini 4 Argon / 3.1 Pro / 3.8 / 3.6 Flash / 3.5 Flash-Lite
    │   ├── kimi.md                Kimi K3.1 Pro / K3 / K2.7-Code / K2.6 (retiring) — forced thinking, Agent Swarms
    │   ├── glm.md                 Z.ai GLM-5.3 / 5.3-Flash / FlashX / 5.2 — reasoning_effort + Claude-Code-router pattern
    │   ├── qwen-frontier.md       Qwen3.8-Max / 3.8-Flash / 3.8-Omni-Flash / 3.7-Max
    │   ├── deepseek.md            V4.1 Flash / V4-Pro + 2026 version lineage — user-prompt-priority pattern
    │   ├── grok.md                xAI Grok 4.7 / 4.6 — the 1M → 500K window regression, `guidelines` param
    │   ├── mistral-frontier.md    Mistral Large 4 / Small 4 / Ministral 14B (+ retiring Large 3, Medium 3.5)
    │   ├── meta.md                Meta Muse Wonder / Spark 1.3 / 1.2 / Spark Open / Glimmer
    │   ├── tencent.md             Tencent Hunyuan Hy4 preview — new family; facts + `?` by design
    │   ├── other-frontier.md      MiniMax M3.1-Flash / Xiaomi MiMo-V2.6 / StepFun Step 5 — no vendor guides
    │   ├── _universal.md          single-vendor universals + cross-vendor (3-way and 4+)
    │   └── small-local.md         Gemma 3/4, small Qwen, Ministral 3, Phi-4-mini, Llama 3.2, Granite 4.2, MiniCPM5,
    │                              Nemotron Dory, LiquidAI LFM, fine-tunes saiga / T-lite / Hermes / HORROR-Imatrix
    │
    ├── archive/                   legacy models — not maintained; load only for a prompt that still targets one
    │
    └── agentic-systems/           system-specific deltas (Class 1)
        ├── claude-code.md         CLAUDE.md hierarchy, @import, skills, subagents, hooks
        ├── codex.md               AGENTS.md hierarchy, override pattern, 32 KiB cap, codex exec
        ├── gemini-cli.md          GEMINI.md, thinking_level, response_json_schema
        └── _common.md             AGENTS.md cross-tool standard + cross-vendor routers
```

---

## Contributing — adding a new model or vendor

See [CONTRIBUTING.md](CONTRIBUTING.md). Short version:

1. **Add a row** to `references/matrix.md` (all 5 tables: A through E). Use `?` for unknowns; honest is better than guessed.
2. **Add a vendor model file** in `references/models/<vendor>.md` if quirks exceed one cell per table.
3. **Add the model to `references/lineup.md`** (and a checklist to `references/model-audits.md` if the vendor ships a delta list); new vendor signals go in SKILL.md Step 2b. Move what it makes two generations stale to `references/archive/`.
4. **Update CHANGELOG.md** with the date, vendor, and what changed.
5. **Cite sources** in the new model file's "Source notes" section — multiple independent sources where available.

The matrix-citation method depends on auditable claims. Don't add a row from memory — cite vendor docs, benchmarks, or independent reviews.

---

## Update cadence

Vendors release new versions frequently. Update files when:

**Class 1 / Frontier:**
- New universal principle → `references/principles.md`
- New model behavior or new model version → matrix row + deltas in `references/models/<vendor>.md`
- New agentic system (Cursor, Aider, Windsurf, etc.) → add `references/agentic-systems/<system>.md` + matrix row
- New technique snippet → `references/techniques.md`
- New anti-pattern → `references/antipatterns.md`

**Class 2 / Small local:**
- New small model family/version → `references/matrix-small.md` + vendor section in `references/models/small-local.md`
- New small-model technique (verified on a suite, not theory) → `references/techniques-small.md`
- New regression with reproducible measurement → `references/antipatterns-small.md`

**Both classes:**
- New multilingual finding → `references/multilingual.md`

Each layer is independent: `principles.md` changes rarely, matrix files grow row-by-row, system files grow when the skill expands its tool coverage.

---

## Uninstall

```bash
rm -rf ~/.claude/skills/prompt-atlas
# or if symlinked
rm ~/.claude/skills/prompt-atlas
```

---

## License

[Creative Commons Attribution-ShareAlike 4.0 International](LICENSE). You're free to use, adapt, and redistribute. Derivative works must carry the same license. Attribution required (link back to the source repo).

The methodology and curated vendor-behavior data are the primary contributions; CC-BY-SA protects against closed-source forks of the knowledge base while letting anyone build commercial tools that *use* it.
