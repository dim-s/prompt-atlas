# Archive — legacy model coverage

Coverage for models that are no longer part of the live atlas, kept verbatim as it stood when archived. **Not maintained** — facts here may be stale, and nothing here is re-checked by the nightly reconciliation.

**When to load it:** only when the prompt under review still targets one of these models, or when migrating a prompt off one (the archived section usually holds the baseline the migration starts from). For every other review, ignore this folder.

**What goes here:** a model is archived when its vendor retired or deprecated it, or when it is two or more generations behind the current lineup and no current-model section is written as a delta against it. A model that is still callable but scheduled to retire stays live with a "retiring <date>" note until the date passes. Several archived models are still callable on their vendor's API (Anthropic lists Opus 4.7 / 4.8, Sonnet 4.6 and Haiku 4.5 as "Legacy — still available"); archiving them is an editorial choice to keep the live reference lean, not a claim that they are gone.

| File | Holds |
|---|---|
| `matrix-legacy.md` | Legacy rows of matrix Tables A–D |
| `models/claude.md` | Opus 4.7 (covers Opus 4.8), Sonnet 4.6, Haiku 4.5 |
| `models/gpt.md` | GPT-5.5, 5.4, 5.3 / 5.3-Codex, 5.2, 5.1 |
| `models/gemini.md` | Gemini 3 Flash, 3.7 Flash, 3.5 Flash (+ Cyber), 3.1 Flash-Lite, 2.5 Pro / Flash |
| `models/kimi.md` | Kimi K2.5, K2 |
| `models/glm.md` | GLM-5.1, GLM-5, GLM-4.6 |
| `models/qwen-frontier.md` | Qwen3.7 Plus, 3.6 Plus, 3.6-Max-Preview, Qwen3-Max-Thinking |
| `models/deepseek.md` | DeepSeek V4-Flash, V4-Flash-Vision-Exp, V3.2 / V3.2-Speciale, R1 |
| `models/grok.md` | Grok 4.5, Grok 4.3 |

Archived in v2.0.0 (2026-10-31). Later archivals append to the same files; `CHANGELOG.md` records each move.
