# Changelog

All notable changes to **prompt-atlas** are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/), and the project adheres to [Semantic Versioning](https://semver.org/) where feasible (model-coverage additions are minor versions; methodology changes are major).

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
