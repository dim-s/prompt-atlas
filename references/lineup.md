# Model lineup — ids, status, reasoning knob, where to read

State as of **2026-10-31**. One row per model the atlas covers; the vendor files hold the prompting guidance, this file holds the facts needed to pick the right section and to know which knob exists. **Knob values are runtime parameters — surface them to the user, never write them into a prompt** (`SKILL.md` § Reasoning-depth knobs). Models two generations behind or retired by the vendor live in `archive/` (`archive/README.md`); a row here that points into `archive/` only says where the section went.

When a row and a vendor file disagree, the vendor file wins (it carries the source); fix the row in the same change.

## Class 1 — frontier

### Anthropic — Claude

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Claude Fable 5.1 (`claude-fable-5-1`); Mythos 5.1 (`claude-mythos-5-1`, Project Glasswing only) | current top tier, Sep 2026 | `effort` low…max, default `high`; adaptive thinking always on — cannot be disabled | claude.md § Claude Fable 5.1 |
| Claude Opus 5.5 (`claude-opus-5-5`) | current Opus, 2026-09-22 | `effort` low…max, default `medium`; **thinking cannot be disabled** (`disabled` / budgets → 400) | claude.md § Claude Opus 5.5 (on § Opus 5 baseline) |
| Claude Sonnet 5.5 (`claude-sonnet-5-5`) | current Sonnet, 2026-09-28 | `effort` low…max, API default `high` (Claude Code / apps `medium`); `disabled` → 400, lowest mode `between_tools` at ≤ `high` | claude.md § Claude Sonnet 5.5 |
| Claude Haiku 5.5 (`claude-haiku-5-5`) | current Haiku, 2026-10-07 | `effort` low…max, default `medium` (first Haiku with effort); `disabled` allowed at ≤ `high`; `between_tools` → 400 | claude.md § Claude Haiku 5.5 |
| Claude Fable 5 / Opus 5 / Sonnet 5 | previous generation (vendor: Legacy, still available) — the baselines the 5.x deltas are written against | Fable 5: always on; Opus 5: default `high`, `disabled` at ≤ `high`; Sonnet 5: default `high`, `disabled` allowed | claude.md § Fable 5 / § Opus 5 / § Sonnet 5 |

### OpenAI — GPT

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| GPT-6 Astra (`gpt-6-astra`) | current, top; system card 2026-09-03 | `reasoning.effort` / `reasoning_effort` low/medium/high/xhigh/max; no default named on model page; cannot be disabled (no `none`) | gpt.md § GPT-6 family, § GPT-6 Astra |
| GPT-6.1 Sol (`gpt-6.1-sol`) | current, released 2026-09-29 | same param; low/medium/high/xhigh/max, default `medium`; no `none` / `minimal` — cannot be disabled | gpt.md § GPT-6.1 Sol and GPT-6.1 Terra |
| GPT-6.1 Terra (`gpt-6.1-terra`) | current, released 2026-10-26 | same param; none/low/medium/high/xhigh/max, default `medium`; disable with `none` | gpt.md § GPT-6.1 Sol and GPT-6.1 Terra |
| GPT-6 Sol (`gpt-6-sol`) | current (live, superseded by 6.1 Sol), 2026-09-22 | same param; none … max, default `medium`; disable with `none` | gpt.md § GPT-6 Sol and GPT-6 Luna |
| GPT-6 Luna (`gpt-6-luna`) | current, 2026-09-22 | same param; none … max, default `medium`; disable with `none` | gpt.md § GPT-6 Sol and GPT-6 Luna |
| GPT-Live 1 (`gpt-live-1`) | current, GA 2026-09-10 (voice, Live endpoint) | none documented on the model page (backend model carries reasoning) | gpt.md § GPT-Live 1 |
| GPT-5.6 Sol / Terra / Luna (`gpt-5.6-sol` / `-terra` / `-luna`) | previous generation, 2026-07-09 (press date); not deprecated in API | `reasoning_effort` none/low/medium/high/xhigh/max, default `medium` (reasoning guide); disable with `none` | gpt.md § GPT-5.6 |

Not models: GPT-6.1 Astra and GPT-6.1 Luna do not exist (404). GPT-6 Rosalind (`gpt-6-rosalind`, 2026-10-30, trusted access, life sciences) is out of scope.

### Google — Gemini

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Gemini 4 Argon (`gemini-4-argon`) | current Pro tier; GA 2026-10-13 (stable; not on Free tier) | `thinking_level` low/medium/high, default `medium`; no `minimal`, no off level listed | gemini.md § Gemini 4 Argon |
| Gemini 3.1 Pro (`gemini-3.1-pro-preview`) | previous Pro tier; preview since 2026-02-19, no shutdown date | `thinking_level` low/medium/high, default `high`; no off level | gemini.md § Gemini 3.1 Pro |
| Gemini 3.8 Flash (`gemini-3.8-flash`) | current Flash; GA 2026-09-02, no shutdown date; also serves `gemini-3.7-flash` requests since 2026-10-08 | `thinking_level` low/medium/high, default `medium`; `minimal` → error; no off level | gemini.md § Gemini 3.8 Flash |
| Gemini 3.6 Flash (`gemini-3.6-flash`) | previous Flash ("previous-generation Flash"); GA 2026-07-21; also serves `gemini-3.5-flash` requests since 2026-10-08 | `thinking_level` minimal/low/medium/high, default `medium`; lowest is `minimal` (off not documented) | gemini.md § Gemini 3.6 Flash |
| Gemini 3.5 Flash-Lite (`gemini-3.5-flash-lite`) | current Lite / subagent tier; GA 2026-07-21 | `thinking_level` minimal…high, default `minimal` | gemini.md § Gemini 3.5 Flash-Lite |
| Gemini 3.7 Flash / 3.5 Flash (`gemini-3.7-flash`, `gemini-3.5-flash`) | deprecated 2026-10-08; endpoints auto-route to 3.8 Flash / 3.6 Flash, no shutdown | n/a (runs as the target model) | archive/models/gemini.md |
| Gemini 3.1 Flash-Lite (`gemini-3.1-flash-lite`) | archived; earliest shutdown 2027-05-07 → 3.5 Flash-Lite | not in the current thinking table (`?`) | archive/models/gemini.md |
| Gemini 2.5 Pro / Flash / Flash-Lite | not deprecated; access limited to past users since 2026-09-18 | `thinking_level` low/medium/high (thinking table); 2.5 Flash-Lite default off | archive/models/gemini.md |
| Gemini 3.8 Flash Cyber | closed access (Fairwind Program) | n/a | gemini.md § Gemini 3.8 Flash Cyber |

### Moonshot — Kimi

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Kimi K3.1 Pro (`kimi-k3.1-pro`, `kimi-k3.1-pro-highspeed`) | current flagship; GA API 23–24.10.2026; weights promised by 04.11 | `reasoning_effort` low/high/max, default max; **cannot be disabled**; sampling fixed | kimi.md § Kimi K3.1 Pro |
| Kimi K3 (`kimi-k3`) | previous flagship; API 16.07.2026, weights 26.07 (press dates) | `reasoning_effort` low/high/max, default max; cannot be disabled on Moonshot's API (QwenCloud-hosted copy documents a `false` switch) | kimi.md § Kimi K3 |
| Kimi K2.7-Code (`kimi-k2.7-code`, `-highspeed`) | live, closed to new users 31.10.2026, retiring by end Dec 2026 | none — thinking and Preserved Thinking forced on | kimi.md § Kimi K2.7-Code |
| Kimi K2.6 (`kimi-k2.6`) | live, closed to new users 31.10.2026, retiring by end Dec 2026 | `thinking` enabled (default) / disabled; no effort ladder | kimi.md § Kimi K2.6 |

### Z.ai — GLM

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| GLM-5.3 (`glm-5.3`) | current; mid-Aug 2026 (launch 17.08 per coverage); weights on HF 25.08 | `reasoning_effort` low/high/max, default max; **cannot be disabled** (`thinking.type` only `enabled`) | glm.md § GLM-5.3 |
| GLM-5.3-Flash (`glm-5.3-flash`) | current cost/multimodal sibling; 26.08.2026 | same as GLM-5.3; `clear_thinking` flag | glm.md § GLM-5.3-Flash |
| GLM-5.3-FlashX (`glm-5.3-flashx`) | current fast tier (200 tok/s); date not stated by vendor; not on Coding Plan | same as GLM-5.3-Flash | glm.md § GLM-5.3-FlashX |
| GLM-5.3-FlashX-Legal (`glm-5.3-flashx-legal`) | enterprise-only by application (~21.10, inferred from an image path) | same surface as FlashX (by docs page) | glm.md § GLM-5.3-FlashX (one line) |
| GLM-5.2 (`glm-5.2`) | previous; 16.06.2026; still listed | `reasoning_effort` high/max, default max; `none`/`minimal` stop thinking; `thinking` toggle — can be disabled | glm.md § GLM-5.2 |

### Alibaba — frontier Qwen

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Qwen3.8-Max (`qwen3.8-max`; open `Qwen3.8-2.4T-A95B`) | current; GA 03.08.2026, weights 12.08 | `reasoning_effort` low/medium/xhigh, default xhigh; hosted: `enable_thinking` can disable; open 2.4T: thinking-only | qwen-frontier.md § Qwen3.8-Max, rule #3 |
| Qwen3.8-Flash (`qwen3.8-flash`; open `Qwen3.8-Flash-Next`) | current cost tier; 25–27.08.2026 | `reasoning_effort` low/medium/xhigh, default xhigh; `enable_thinking` on by default, can disable (both surfaces); `preserve_thinking` hosted off / open on | qwen-frontier.md § Qwen3.8-Flash, rule #3 |
| Qwen3.8-Omni-Flash (`qwen3.8-omni-flash`) | current, API-only; 18.09.2026 | `reasoning_effort`; hybrid, on by default (raw); default xhigh, `none` disables (secondary) | qwen-frontier.md § Qwen3.8-Omni-Flash |
| Qwen3.7-Max (`qwen3.7-max`) | previous; 20–21.05.2026; still supported (no retirement notice found) | thinking toggle (`enable_thinking`), hybrid on Qwen Cloud | qwen-frontier.md § Qwen3.7-Max |

### DeepSeek

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| V4.1 Flash (`deepseek-flash`; build `DeepSeek-V4.1-Flash-1017`) | current; 2026-09-10, build 1017 since 2026-10-17; also serves legacy `deepseek-v4-flash` / `-vision-exp` | `thinking.type` + `reasoning_effort` (CC) / `output_config.effort` (Anthropic) / `reasoning.effort` (Responses); low/high/max (+aliases minimal, medium, xhigh, ultra); default on at `high`; yes — `thinking.type: disabled` or Responses `none`. Open weights: integer 1–100, default 75 | deepseek.md § DeepSeek V4.1 Flash; rule #5 |
| V4-Pro (`deepseek-v4-pro`; build `DeepSeek-V4-Pro-0813`) | current; preview 2026-04-24, GA 2026-08-13; continued past 2026-09-14 | same as above; default on at `high`; yes | deepseek.md § DeepSeek V4-Pro; rule #5 |
| V4-Flash (`deepseek-v4-flash`, build 0731) | retired; id served by V4.1-Flash-1017 | was low/high/max | archive/models/deepseek.md |
| V4-Flash-Vision-Exp (`deepseek-v4-flash-vision-exp`) | retired (2026-09-10); id served by V4.1-Flash-1017 | was low/high/max | archive/models/deepseek.md |
| V3.2 / V3.2-Speciale; R1 | legacy; V3.2 2025-12-01; `deepseek-chat`/`-reasoner` discontinuation announced for 2026-07-24, not on pricing page | single toggle (chat vs reasoner) | archive/models/deepseek.md |

### xAI — Grok

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Grok 4.7 (`grok-4.7`) | current, recommended; 2026-09-21 | `reasoning_effort` low/medium/high/xhigh, default high; **cannot be disabled** | grok.md § Grok 4.7 |
| Grok 4.6 (`grok-4.6`) | previous; 2026-08-12; callable, no shutdown date | `reasoning_effort` low/medium/high/xhigh, default high; **cannot be disabled** | grok.md § Grok 4.6 |
| Grok 4.5 (`grok-4.5`) | older; callable, no shutdown date (section archived) | `reasoning_effort` low/medium/high, default high (`xhigh` → `high`); **cannot be disabled** | archive/models/grok.md |
| Grok 4.3 (`grok-4.3`) | older; callable, 1M window (section archived) | `reasoning_effort` none/low/medium/high/xhigh, default **low**; **can be disabled** (`none`) | archive/models/grok.md; grok.md rule #1 |

### Mistral — frontier

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Mistral Large 4 (`mistral-large-4` / `mistral-large-4-0`; weights `mistralai/Mistral-Large-4-1T-A52B`) | current flagship; preview 2026-10-06, weights + improved model 2026-10-29; docs status still "Public Preview" | `reasoning_effort` low/medium/high, default high; **cannot be disabled** (no `none`) | mistral-frontier.md § Mistral Large 4 |
| Mistral Large 3 (`mistral-large-2512`) | previous; deprecated 2026-10-31, retiring 2027-04-30 → Large 4 | no per-request knob documented in the atlas | mistral-frontier.md § Mistral Large 3 |
| Mistral Medium 3.5 (`mistral-medium-3-5`) | deprecated 2026-10-31, retiring 2027-04-30 → Large 4 | `reasoning_effort` high/none (high recommended for agentic/code); **can be disabled** (`none` = minimal thinking, chunk omitted) | mistral-frontier.md § Mistral Medium 3.5 |
| Mistral Small 4 (`mistral-small-2603`) | current; 2026-03 | `reasoning_effort` high/none; **can be disabled** (`none`) | mistral-frontier.md § Mistral Small 4 |
| Ministral 3 14B / 8B (2512) | current edge generation | no knob: Instruct vs Reasoning variant | mistral-frontier.md § Ministral 3 |

Ministral 3 8B and smaller are Class 2 (`small-local.md`); 14B is listed here because it is above the Class 2 boundary.

### Meta — Muse

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Muse Wonder 1.0 (`muse-wonder-1.0`) | current, preview; 2026-10-12 | `reasoning_effort` (Chat Completions) / `reasoning.effort` (Responses); minimal/low/medium/high/xhigh/max; omitted = model-determined level; **cannot be disabled** (`none` → 400) | meta.md § Muse Wonder 1.0 |
| Muse Wonder Full (no API id) | Muse Code subscription only (Power / Maximum Usage); 2026-10-12 | via Muse Code `--reasoning-effort`; not selectable on the API | meta.md § Muse Wonder 1.0 |
| Muse Spark 1.3 (`muse-spark-1.3`, `muse-spark-1.3-contributor`) | current hosted Spark ("Recommended for new work"); 2026-09-02 | same knob as Wonder; `max` not on contributor; **cannot be disabled** | meta.md § Muse Spark 1.3 |
| Muse Spark 1.2 (`muse-spark-1.2`) | previous; 2026-08-05; default model in Muse Code | same knob, minimal…xhigh; **cannot be disabled** | meta.md § Muse Spark 1.2 |
| Muse Spark Open (`meta-models/Muse-Spark-Open-816B-A42B`) | current open weights; 2026-10-15 | `reasoning_effort` low/medium/high/xhigh, default xhigh; **cannot be disabled**; a `Reasoning strength: N.` system-prompt line overrides the parameter | meta.md § Muse Spark Open |
| Muse Glimmer (`Muse-Glimmer-30B`) | current open weights; 2026-08-10 | `reasoning_strength` template arg xhigh/high/medium/low, default high; no off value documented | meta.md § Muse Glimmer |

### Tencent — Hunyuan

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Hy4 preview (`tencent/Hy4-preview`; TokenHub / OpenRouter) | current preview; 2026-08-28; no GA as of 31.10 | open weights `chat_template_kwargs.reasoning_effort`: `high` (default) / `no_think`; no ladder; yes (`no_think`); hosted `?` | tencent.md § Tencent Hy4 preview |

### MiniMax

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| M3.1-Flash (`MiniMax-M3.1-Flash`) | current; weights 2026-10-17, API 2026-10-19 | `output_config.effort` / `reasoning_effort` / `reasoning.effort`; low/medium/high/xhigh/max; hosted default `max` (template: none); **no** (400) | other-frontier.md § MiniMax M3.1-Flash |
| M2.5 / M2.7 (+ `-highspeed`) | closed to new users 2026-10-19; retiring 2026-11-18; plan traffic routed to M3.1 Flash | — (not covered) | other-frontier.md § MiniMax M2.5 / M2.7 |

### Xiaomi — MiMo

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| MiMo-V2.6 Pro / Flash / Pro-ultraspeed (`mimo-v2.6-pro`, `mimo-v2.6-flash`, `mimo-v2.6-pro-ultraspeed`) | current; 2026-09-22 (MOPD weights 27.09) | `thinking.type` enabled/disabled (open weights `enable_thinking`); no ladder; default enabled; yes | other-frontier.md § Xiaomi MiMo-V2.6 |
| MiMo-V2.5 / V2.5-Pro | retired 2026-10-21 10:00 UTC+8 (→ V2.6-Flash / V2.6-Pro) | — | other-frontier.md § Xiaomi MiMo-V2.6 (one line) |

### StepFun

| Model (id) | Status + date | Reasoning knob — param, ladder, default, **can it be disabled** | Read |
|---|---|---|---|
| Step 5 Preview (`step-5-preview`) | current preview; HF weights 2026-09-20 | `reasoning_effort` (CC) / `output_config.effort` (Messages); low/medium/high; `medium` recommended, omitted default `?`; no (always thinks) | other-frontier.md § StepFun Step 5 Preview |

MiniMax, Xiaomi MiMo and StepFun publish no prompting guide — all behavioral cells are `?`; one shared file, `models/other-frontier.md`.

## Class 2 — small local (dense 2–9B, or MoE ≤ ~9B active / ≤ ~35B total)

Read `models/small-local.md` § family and `matrix-small.md`. Rule for every row: thinking on a 2–3B single-pass task is a measured regression — switch it off with the template flag where one exists, never with prose.

| Vendor | Model (id) | Status + date | Reasoning knob | Read |
|---|---|---|---|---|
| OpenBMB | MiniCPM5-2B (`openbmb/MiniCPM5-2B`) | current; HF repo 2026-09-06 | `enable_thinking` template kwarg, three states: `True` / `False` / absent = model decides; disable with explicit `False` | small-local.md § OpenBMB MiniCPM5 |
| NVIDIA | Nemotron-Labs Dory 1.0 4B (`nvidia/NVIDIA-Nemotron-Labs-Dory1.0-4B-BF16`, + MXFP8/NVFP4/GGUF) | current; released 2026-10-16 | Depth = `n_recurrent_loops` / `dory.num_loops` 1–4, default 2 (serving). Reasoning "always on in vLLM", llama.cpp `--reasoning on`. Template has undocumented `enable_thinking` (default True), `medium_effort`, `truncate_history_thinking`; disabling is undocumented | small-local.md § NVIDIA Nemotron-Labs Dory |
| LiquidAI | LFM3-1.7B (`LiquidAI/LFM3-1.7B`) | current; HF 2026-10-23, blog 2026-10-27 | `enable_thinking`, default on; `False` "can be inconsistent"; `preserve_thinking` keeps earlier-turn reasoning | small-local.md § LiquidAI LFM |
| LiquidAI | LFM3-Omni-2B / LFM3-VL-2B | current; HF 2026-10-21 / 10-19 | Omni: "non-reasoning", `enable_thinking=True` not supported. VL: thinking (knob not detailed in what was read) | small-local.md § LiquidAI LFM (siblings) |
| LiquidAI | LFM2.5-24B-A2B (`LiquidAI/LFM2.5-24B-A2B`) | current; HF 2026-10-12 | none: "pure reasoning model that always thinks"; cannot be disabled | small-local.md § LFM2.5-24B-A2B |
| JetBrains | Mellum2.1-12B-A2.5B-Thinking | current (MTP variant 2026-10-30) | "a thinking model", `--reasoning-parser qwen3`; no off switch documented | small-local.md § Noted, not profiled |
| IBM | Granite 4.2 3B / 8B (`ibm-granite/granite-4.2-3b` / `-8b`) | current; 2026-08-25 | chat-template mode: thinking (default) / non-thinking / low-effort; can be disabled | small-local.md § IBM Granite |
| IBM | Granite Switch 4.1 8B (`ibm-granite/granite-switch-4.1-8b`) | current; rev 4.1 2026-10-14, 4.1.1 2026-10-27 | adapter picked via `chat_template_kwargs.adapter_name`; chat reasoning knob not documented on the card | small-local.md § IBM Granite (Switch paragraph) |
| Mistral | Ministral 3 3B / 8B (2512), Instruct / Reasoning / Base | current, GA (HF repos 2025) | reasoning = separate `-Reasoning-2512` checkpoints; no per-request knob found in what was read (`?`) | small-local.md § Mistral / Ministral |
| Mistral | Ministral 3B / 8B (2024; `ministral-3b-2410` / `-8b-2410`) | API retired 2025-12-31 | n/a | small-local.md § Mistral / Ministral |
| Google | Gemma 4 E2B / E4B | current; weights unchanged since 2026-07-20 | native thinking via template; atlas rule: keep off (measured −25/−44pp) | small-local.md § Google Gemma |
| Google | Gemma 3 4B | previous | no native thinking | small-local.md § Google Gemma |
| Alibaba | Qwen 3.5 2B / 4B / 8B / 9B | current small (no new small Qwen as of 10-31) | `enable_thinking` True/False, default on; soft switches only work when True | small-local.md § Alibaba Qwen |
| Xiaomi | MiMo-V2.6-Distill-Qwen-9B | noted (Qwen 3.5 9B tune) | `chat_template_kwargs.enable_thinking` | small-local.md § Alibaba Qwen |
| Microsoft | Phi-4-mini 3.8B / Phi-4-mini-reasoning | current small | separate reasoning variant; no knob | small-local.md § Microsoft Phi |
| Meta | Llama 3.2 1B / 3B | current small (newest `meta-llama` repo 2025-04) | none | small-local.md § Meta Llama |

