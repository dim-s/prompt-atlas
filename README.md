# prompt-atlas

An auditable, citation-driven skill for **reviewing and improving the wording of LLM prompts** across all major current model families. Every recommendation cites a documented behavior (model × axis cell) so the user can challenge it from data, not authority.

License: [CC-BY-SA 4.0](./LICENSE). Built on the methodology of [matrix-citation prompt review](references/matrix.md).

---

## Coverage

### Class 1 — Frontier agent-facing prompts (9 vendor families)

| Vendor | Models covered |
|---|---|
| **Anthropic Claude** | Opus 4.7 / Sonnet 4.6 / Haiku 4.5 + legacy 4.6 |
| **OpenAI GPT-5.x** in Codex CLI | GPT-5.5 (+ Instant variant) / 5.4 / 5.3 / 5.3-codex / 5.2 / 5.1 |
| **Google Gemini 3.x** in Gemini CLI | 3.1 Pro / 3 Flash / **3.5 Flash** (May 2026) / 3.1 Flash-Lite + 2.5 legacy |
| **Moonshot Kimi** | K2.6 / K2.5 / K2 (retires 2026-05-25) |
| **Z.ai GLM** | GLM-5.1 / GLM-5 / GLM-4.6 |
| **Alibaba Qwen frontier** | Qwen3.7-Max / 3.7 Plus / 3.6 Plus / 3.6 Max-Preview / 3-Max-Thinking |
| **DeepSeek** | V4-Pro / V4-Flash / V3.2 / R1 |
| **xAI Grok** | Grok 4.3 (Grok 5 pending Q2 2026) |
| **Mistral frontier** | Mistral Large 3 / Mistral Small 4 / Ministral 3-8B+ reasoning |

Covers CLAUDE.md, AGENTS.md (Claude Code / Codex / Gemini variants), GEMINI.md, AGENTS.override.md, vendor-specific overrides (`AGENTS.deepseek.md`, `AGENTS.glm.md`, etc.), SKILL.md, subagent definitions, slash commands, ad-hoc prompts including headless `codex exec` / Gemini CLI invocations.

### Class 2 — Small local task-facing prompts (2-9B)

- **Google Gemma** 3 (1B / 4B / 12B / 27B) and Gemma 4 (e2b / e4b)
- **Alibaba Qwen** 3.5 (2B / 4B / 8B / 9B) — small variants only
- **Mistral / Ministral** 3B / 7B / Nemo 12B — small variants only
- **Microsoft Phi** 4-mini (3.8B) / Phi-4-mini-reasoning
- **Meta Llama** 3.2 (1B / 3B)
- **Notable fine-tunes** — saiga (Russian Gemma), T-lite (Yandex), Hermes (Nous tune of Llama), HORROR-Imatrix, TrevorJS uncensored variants

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

API/SDK parameter values (`temperature`, `max_tokens`, `reasoning_effort`, `text.verbosity`, `thinking_level`, `extra_body.thinking`, `thinking: "off"|"high"|"max"`, `enable_thinking` value choice), Codex `config.toml`, MCP server registration, hook scripts, Python/TS code around the SDKs, LoRA / SFT training data design. Those belong to vendor docs or other specialized skills.

The skill **surfaces** non-wording levers (e.g., "raise `effort` before rewriting", "switch to a different quant") as conversation points when relevant — but never embeds them into the artifact under review.

---

## Methodology — matrix citation

Every finding cites a specific row × column from [references/matrix.md](references/matrix.md) (frontier) or [references/matrix-small.md](references/matrix-small.md) (small local). Example finding:

> `[IMPROVE]` Persona line "You are a senior engineer" on a cross-vendor `AGENTS.md`.
> **Why:** matrix.md Table A column "Persona / 'act as'" — Gemini 3 expects +5% boost, GPT-5.5 hurts, GLM-5.1 OK but no identity-pinning. Cross-vendor compromise: use methodological anchor (techniques.md §6) rather than credential persona.

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

- **Codex CLI** — place under `.codex/skills/prompt-atlas/` or reference from project `AGENTS.md`
- **Gemini CLI** — place under `.gemini/skills/prompt-atlas/` or reference from `GEMINI.md`

The matrix-citation method works the same way in either host.

---

## Verify it's loaded

In a Claude Code session, ask any of these:

**Class 1 (frontier):**
> "проверь мой CLAUDE.md" (or paste any CLAUDE.md-style text)
> "адаптируй этот промпт под GPT-5.5 / Codex"
> "review my AGENTS.md for cross-vendor use"
> "tune for Gemini 3.5 Flash"
> "почему GLM не думает в Claude Code?"
> "адаптируй под DeepSeek V4"

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
    ├── matrix.md                  frontier model × axis (9 vendors × 5 tables)
    ├── artifacts.md               per-artifact checklists — CLAUDE.md / AGENTS.md / SKILL.md /
    │                              subagent / slash-command / ad-hoc
    ├── techniques.md              frontier wording snippets — outcome-first, output contracts,
    │                              creative kernel, anti-hallucination, etc.
    ├── antipatterns.md            frontier anti-patterns with fixes
    │
    ├── # Class 2 — Small local task-facing
    ├── matrix-small.md            small-model × axis — Gemma / small Qwen / Ministral / Phi / Llama
    ├── techniques-small.md        small-model snippets — skeleton, few-shot-at-end, named anti-patterns
    ├── antipatterns-small.md      12 anti-patterns specific to small models with reproduced regressions
    │
    ├── multilingual.md            non-EN prompt handling — applies to BOTH classes
    │
    ├── models/                    vendor-specific deltas
    │   ├── claude.md              Opus 4.7 / Sonnet 4.6 / Haiku 4.5
    │   ├── gpt.md                 GPT-5.1 → 5.5 + Instant variant
    │   ├── gemini.md              Gemini 3.1 Pro / Flash / 3.5 Flash / Flash-Lite
    │   ├── kimi.md                Kimi K2.6 / K2.5 / K2 — Agent Swarms protocol
    │   ├── glm.md                 Z.ai GLM-5.1 / 5 / 4.6 — Claude-Code-router pattern
    │   ├── qwen-frontier.md       Qwen3.7-Max / 3.6 Plus / Max-Preview / 3-Max-Thinking
    │   ├── deepseek.md            V4-Pro / V4-Flash / V3.2 — user-prompt-priority pattern
    │   ├── grok.md                xAI Grok 4.3
    │   ├── mistral-frontier.md    Mistral Large 3 / Small 4 / Ministral 3-8B+
    │   ├── _universal.md          single-vendor universals + cross-vendor (3-way and 4+)
    │   └── small-local.md         Gemma 3/4, small Qwen, small Ministral, Phi-4-mini, Llama 3.2,
    │                              fine-tunes saiga / T-lite / Hermes / HORROR-Imatrix
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
3. **Update SKILL.md** Step 2b (vendor signals) and Step 2c (model version options) so the workflow recognizes the new target.
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
