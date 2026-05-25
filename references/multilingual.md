# Multilingual prompts — when target language is not English

Most prompt-engineering literature assumes the user wants English output. This file covers the systematic ways things break when the prompt or the expected output is in another language — Russian, French, Japanese, etc. — and the techniques that survive contact with research as of 2026.

## When to read this reference

Trigger this file in addition to the core workflow whenever:

- The prompt body is written in a non-English language
- The prompt explicitly instructs the model to respond in a non-English language
- The user complains about "англицизмы", "смешение языков", "model keeps switching to English", "vocabulary leakage", "слишком много английских слов", "лексика не та", "теряет регистр"
- The artifact targets a regional / localized agent (Russian-language coach, German legal advisor, Japanese narrative editor)
- A subagent reads non-English input from users but its system prompt is English (common silent failure mode)

---

## The core problem — code-switching is a latent feature, not an input bug

The folk model says: "the prompt was unclear, so the model mixed languages". The 2025–2026 research says the opposite.

**Code-switching concentrates in the model's final layers and reflects internal processing preferences, not surface input.** Two independent papers establish this:

- **CoCo-CoLa** (Razzaghi et al., Feb 2025, [arxiv:2502.12476](https://arxiv.org/abs/2502.12476)): *"final layers play a crucial role in determining output language"*. Task knowledge and language choice are **separable** — the model "knows the answer" but picks the wrong output language. This is why "respond in Russian" pushed late in the prompt often works while "respond in Russian" buried at the top often doesn't.

- **Language Mixing in Reasoning LMs** (Li et al., May 2025, [arxiv:2505.14815](https://arxiv.org/pdf/2505.14815)): *"the script composition of reasoning traces closely aligns with that of the model's internal representations, indicating that language mixing reflects latent processing preferences in RLMs"*. Reasoning models think in a default script (usually Latin); the visible mixing is a leak of that internal state.

**The "English gravity well":** large-corpus English skew means the model implicitly translates abstract concepts to English mid-computation, then translates back at output. Documented in pretraining-mix research ([arxiv:2510.25947](https://arxiv.org/abs/2510.25947)) and explicitly named in the multilingual-consistency line of work ([arxiv:2509.23659](https://arxiv.org/abs/2509.23659), EMNLP 2025): *"up to 29% accuracy drop in non-English languages compared to English"*.

**Specific bias against Russian** is documented in [arxiv:2604.07123](https://arxiv.org/abs/2604.07123) (Östling & Kurfalı, Apr 2026, "Language Bias under Conflicting Information in Multilingual LLMs") — across models tested, *"a general bias against Russian"* in resolution patterns. Practical consequence: Russian-language adherence and lexical purity are noticeably worse than for French/German/Spanish at comparable corpus exposure.

## What this means for prompt-atlas

- Treat code-switching reports as **architecture-level**, not "the user wrote a bad prompt". Don't promise full elimination via wording alone.
- Surface the **non-prompt levers** first (separate thinking-language from output-language; verify the user isn't running with `effort: low` on a creative task; ensure the system language matches expected output language).
- Then propose the wording fixes below — they help, but ceiling is real.

---

## Model-by-model picture (May 2026)

**No public benchmark measures "anglicism rate in Russian output" by model.** This is a research gap. The picture below stitches vendor docs, community reports, and adjacent academic data.

### Claude Opus 4.7 / Sonnet 4.6 / Haiku 4.5

- Anthropic [migration guide](https://platform.claude.com/docs/en/docs/about-claude/models/migrating-to-claude-4) warns: *"Claude Opus 4.7 is more direct and opinionated, with less validation-forward phrasing... If your product relies on a specific voice, re-evaluate style prompts against the new baseline."* This affects multilingual voice too — softer 4.6 style instructions for non-English voice often go ignored on 4.7. Migrating prompts: re-test register adherence.
- Tokenizer rebuilt on 4.x — Anthropic claims efficiency gains for non-Latin scripts. Doesn't fix lexical drift.
- No vendor documentation specifically on code-switching or anglicism control. Slot in for migration: explicit style anchor + final-position language instruction.
- Sonnet 4.6 / Haiku 4.5 — no specific public data on language adherence. Extrapolating from CoCo-CoLa: smaller / distilled models drift to English defaults more than the flagship.

### GPT-5.5 / 5.4 / 5.3 (Codex CLI and API)

- **OpenAI's [GPT-5.5 prompting guide](https://cdn.openai.com/API/docs/gpt-5-5_prompting_guide.pdf) and [Codex prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) do not mention multilingual behavior at all.** Notable silence — treat any cross-language requirement on GPT-5.x as user-implemented from scratch.
- Tokenizer rebuilt; non-Latin scripts more efficient than GPT-4.x. Doesn't fix lexical drift.
- 5.5 is more aggressive about pruning "redundant" instructions — language instructions that are repeated may be dropped by the model's own attention budgeting. Counter: put the language constraint where it matters (final position) and don't dilute it with stylistic adjectives in three places.

### Gemini 3.1 Pro / Flash / Flash-Lite

- **Most concrete public evidence of language mixing.** [Google gemini-cli issue #13715](https://github.com/google-gemini/gemini-cli/issues/13715) is a master issue consolidating reports of *"Multiple languages and alphabet appear in the CLI output"*. The merged duplicates explicitly include **"Cyrlic/russian"** and Arabic-English mixing. **Status: closed as not planned.** Google does not have a fix in 2026.
- Google's [Gemini 3 prompting best practices](https://www.philschmid.de/gemini-3-prompt-practices) does not address language mixing.
- `thinking_level: high` (default for 3.1 Pro) — adaptive thinking can leak Latin-script reasoning into output. For non-English output where reasoning isn't required: drop to `low` or `minimal` and rely on adherence improving.

### Cross-vendor universal

Worst-case constraint. The compromise from `models/_universal.md` plus this file: explicit language tag, style anchor, whitelist-not-blacklist, language instruction repeated at the end. Avoid vendor-specific tricks (XML tags work on Claude/Gemini but Codex is markdown-first).

---

## What works — techniques that survived research contact

Each technique below is tagged with how strong the evidence is.

### 1. Style anchor — show, don't tell ⭐ strongest

The single highest-leverage technique for non-English output on current models, especially Opus 4.7 (which is more literal than 4.6 and copies the register of provided examples more faithfully). One short paragraph in target voice beats any paragraph of adjectives.

**Snippet (RU example, adapt to user's language):**

```
Стилистический ориентир — пиши в этом регистре:

> [пользователь вставляет 1–3 абзаца своего эталонного текста или
> текста референсного автора в нужном регистре]

Если расхождение между этим примером и моим запросом — следуй примеру.
```

Why this works on 4.7 specifically: tokenized literally, treated as a distribution to mirror — bypasses the "describe voice in words" failure mode where adjectives like "warm" / "тёплый" / "академический" get under-weighted.

### 2. Whitelist English terms, do NOT blacklist ⭐ strong

The pink-elephant problem is documented: *"LLMs are really bad at following negative instructions... Token generation inherently leans toward positive selection"* (Gadlet, [synthesis of Anthropic + OpenAI guidance](https://gadlet.com/posts/negative-prompting/)). Long blacklists ("don't write деплой, don't write фронтенд") prime the model to use exactly those words.

Inverse the framing: declare which English is **allowed**, treat the rest as Russian-by-default.

**Snippet (RU example):**

```
Английскими оставляй:
- Идентификаторы кода (имена функций, переменных, классов)
- Названия библиотек, инструментов, протоколов (React, Postgres, HTTP)
- Общепринятые сокращения (API, JSON, CLI, SDK)

Для всего остального — естественные русские слова.
Если устойчивого русского эквивалента нет — оставь английский термин,
не выдумывай кальку.
```

The last clause is critical. Without it, the model invents awkward calques to obey the rule. With it, untranslatable terms stay English and the rule holds for the rest.

### 3. Separate thinking-language from output-language ⭐ strong, with research data

Forcing the model to think in the output language costs accuracy. [arxiv:2505.22888](https://arxiv.org/abs/2505.22888) ("When Models Reason in Your Language") measures this on AIME with DeepSeek-Distilled-R1-32B: prompt-hacking language matching from ~45% to ~98% **drops accuracy from 25.5% to 17.0%** — a 33% relative loss.

Russian is the bright spot: *"Russian demonstrates notably stronger performance than other non-English languages... the model achieved approximately 100% accuracy in actually producing Russian thinking traces"*. But the trade-off principle holds — for reasoning-heavy work, let thinking happen in any language, gate only the output.

**Snippet (RU example):**

```
Можешь рассуждать на любом удобном тебе языке — английском,
русском, смешанно. Финальный ответ — только на русском.
```

For Claude / GPT-5.x with extended thinking, this routes the cost-of-multilingualism onto invisible thinking tokens and keeps the visible answer clean.

**When NOT to apply:** pure stylistic / rewriting / editorial tasks where thinking IS the output (the model writing prose). Force the entire pipeline into the target language and accept the smaller accuracy hit — it's mostly about register, not reasoning.

### 4. Final-position language instruction ⭐ moderate, theoretical grounding

CoCo-CoLa locates language choice in the final layers. Operationally, the last instruction the model reads before generating has the strongest influence on output language. Long system prompts that mention language only at the top routinely fail.

**Snippet:**

End the system prompt with one terse line:

```
[Last line of prompt]
Reply in Russian.
```

For multilingual artifacts where vendor allows: pair with a tag.

```
<output_language>ru</output_language>
```

Tag works well on Claude and Gemini (XML-tolerant). Markdown header works on Codex (which prefers prose).

### 5. System prompt language matches target output language ⭐ moderate

Documented in [transformer-circuits.pub](https://transformer-circuits.pub/) work on Claude — the model reads the system prompt language as a "hypothesis about the user" and tunes voice / register accordingly. A fully English system prompt instructing "respond in Russian" creates conflict; the model treats English as the dominant context.

**Action:** if the agent is permanently Russian-facing, write the system prompt itself in Russian, not just the instruction "respond in Russian".

**Exception:** trigger phrases inside `description:` fields of skills / subagents — keep English keywords alongside the localized phrasing because Claude Code / Codex / Gemini CLI heuristics match English triggers more reliably. (Same caveat as in core SKILL.md § Language.)

### 6. Register choice — ask, don't assume ⭐ practical insight

The asymmetry of training corpora (English dominant, much technical Russian content itself written by industry insiders who use anglicisms natively) means **some "anglicisms" are the native register, not the failure**. "Деплой / фронтенд / коммит" is normal in modern Russian dev-community writing. Treating those as drift to be eliminated forces the model into an unnatural academic register the user may not want.

**Action for prompt-atlas:** when a user asks "убери англицизмы", clarify which register they want:

- **Академический / редакторский** — "развёртывание", "интерфейсная часть", "фиксация изменений". Use for documentation, formal writing.
- **Нейтральный технический** — "развёртывание / деплой" (mixed-tolerant), "frontend / фронтенд". Use for mid-formality docs.
- **Разговорный технический** — "деплой", "фронт", "коммит". Native register for dev blogs and chats; "fixing anglicisms" here is fighting the audience.

Apply the rest of the techniques against the chosen register, not against an imaginary "anglicism-free" target.

---

## Antipatterns

### A1. Hard prohibition "no English words" — breaks code, invents calques

```
❌ Никогда не используй английские слова в ответе.
```

Causes (documented): broken identifiers in code blocks, invented translations ("искусственный интеллект" forced where API context wants "AI"), comically literal calques. Replace with Whitelist (technique #2 above).

### A2. Long negative example list — primes mimicking

```
❌ Не пиши:
- deployment → НЕ деплоймент, а развёртывание
- endpoint → НЕ эндпойнт, а конечная точка
- pipeline → НЕ пайплайн, а конвейер
- ... [еще 40 строк]
```

[Contrastive in-context learning research](https://arxiv.org/abs/2401.17390) and folk-knowledge synthesis at [eval.16x.engineer](https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis): more than ~2-3 negative examples and the model starts pattern-matching the bad side. Cap negative pairs at 2-3, frame them as "general principle to extrapolate".

### A3. English instruction inside English system prompt asking for Russian output

```
❌ [Long English system prompt]
You are a senior Russian-speaking advisor. Respond only in Russian.
```

Conflicting context: the model has read 800 English tokens and is being asked, in English, to suddenly switch. Adherence sags. Fix per technique #5: write the system prompt in Russian, or at minimum end with a tagged language declaration.

### A4. High `effort` / `thinking_level` on pure stylistic tasks

Reasoning-trace research ([arxiv:2505.14815](https://arxiv.org/pdf/2505.14815)) shows reasoning models lean to Latin-script thinking; longer traces leak more. For rewriting / editing tasks where there's no real reasoning to do, drop `effort` to `medium` or `low` — fewer thinking tokens, less leakage. (This is a non-prompt lever; surface it to the user, see core SKILL.md § Effort.)

### A5. Promising elimination

```
❌ Этот промпт уберёт все англицизмы.
```

The research is clear that code-switching is a latent property of the model. Wording techniques shift it; they don't eliminate it. Frame recommendations as "significantly reduce" / "shift register" — don't oversell.

---

## Asymmetry: Russian ↔ English is not symmetric

Anglicism leakage in Russian output >> Russian leakage in English output. Causes:

- Corpus skew (the documented "general bias against Russian" in [arxiv:2604.07123](https://arxiv.org/abs/2604.07123); English-pivot internal representation in multilingual surveys).
- English is the LLM's lingua franca for abstract operations; Russian gets pulled toward it.
- Technical Russian corpora themselves are anglicism-heavy (real-world register, faithfully reproduced by the model).

Operational consequence: a "respond in Russian without anglicisms" instruction is **fighting both the model's internal representation AND the dominant register in its training data**. Style anchor + register choice (techniques #1 and #6) work because they reframe the goal — pick a register that exists in the corpus, then steer to it. They don't ask the model to invent a register it hasn't seen.

---

## Quick-reference snippets, copy-adapt

For the Findings / Changes section of a prompt-atlas review.

**Style anchor — paste at end of system prompt:**
```
Стилистический ориентир — пиши в этом регистре:
> [1–3 абзаца эталонного текста]
```

**Whitelist English:**
```
Английскими оставляй: идентификаторы кода, имена библиотек/инструментов,
устойчивые сокращения (API, JSON, HTTP). Для остального — естественные
русские слова. Нет русского эквивалента — оставь английский, не выдумывай.
```

**Separate thinking / output language:**
```
Рассуждать можешь на любом языке. Финальный ответ — только на русском.
```

**Final-position language gate (always end with this):**
```
<output_language>ru</output_language>
Reply in Russian.
```

**Register declaration (pair with Style anchor):**
```
Регистр: [академический редакторский / нейтральный технический / разговорный технический].
В нейтральном регистре устойчивые англицизмы ("деплой", "фронтенд") допустимы.
```

---

## Sources

Verified May 2026. All arxiv links confirmed to resolve to a real paper with the stated content.

**Research (primary):**
- [CoCo-CoLa: Evaluating and Improving Language Adherence in Multilingual LLMs (Feb 2025)](https://arxiv.org/abs/2502.12476) — final-layer language locus, English-tuning drops adherence by 30+ points.
- [Language Mixing in Reasoning Language Models: Patterns, Impact, and Internal Causes (May 2025)](https://arxiv.org/abs/2505.14815) — first systematic study of mixing in RLMs, 15 languages.
- [When Models Reason in Your Language: Controlling Thinking Language Comes at the Cost of Accuracy (May 2025)](https://arxiv.org/abs/2505.22888) — quantifies the language-match vs accuracy trade-off; Russian shown as easier-case.
- [Language steering in latent space to mitigate unintended code-switching (Oct 2025)](https://arxiv.org/abs/2510.13849) — PCA-based inference-time mitigation, 63–99% code-switching reduction.
- [Aligning LLMs for Multilingual Consistency in Enterprise Applications (Sep 2025, EMNLP 2025)](https://arxiv.org/abs/2509.23659) — batch-wise multilingual fine-tuning; documents 29% non-English accuracy gap.
- [Revisiting Multilingual Data Mixtures in Language Model Pretraining (Oct 2025)](https://arxiv.org/abs/2510.25947) — pretraining mix effects; English as pivot benefits multiple families.
- [Language Bias under Conflicting Information in Multilingual LLMs (Apr 2026)](https://arxiv.org/abs/2604.07123) — documents general bias against Russian across tested models.
- [PingPong: A Natural Benchmark for Multi-Turn Code-Switching Dialogues (Jan 2026)](https://arxiv.org/abs/2601.17277) — natural human-authored code-switched dialogues; SOTA models perform poorly.
- [Customizing LM Responses with Contrastive In-Context Learning (AAAI 2024)](https://arxiv.org/abs/2401.17390) — negative-example methodology, useful for understanding limits of blacklist approach.

**Vendor docs:**
- [Anthropic Claude 4 migration guide](https://platform.claude.com/docs/en/docs/about-claude/models/migrating-to-claude-4) — 4.7 voice changes; no multilingual section.
- [OpenAI GPT-5.5 prompting guide](https://cdn.openai.com/API/docs/gpt-5-5_prompting_guide.pdf) — no multilingual section.
- [OpenAI Codex prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) — no multilingual section.
- [Gemini 3 prompting best practices (Phil Schmid)](https://www.philschmid.de/gemini-3-prompt-practices) — no multilingual section.

**Public bug reports:**
- [Google gemini-cli issue #13715 (closed as not planned)](https://github.com/google-gemini/gemini-cli/issues/13715) — master issue consolidating multi-script output reports including Cyrillic/Russian.

**Synthesis / commentary (lower authority, useful framing):**
- [eval.16x.engineer — pink elephant analysis](https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis)
- [Gadlet — why positive prompts outperform negative](https://gadlet.com/posts/negative-prompting/)
