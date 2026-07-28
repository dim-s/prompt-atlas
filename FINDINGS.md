# FINDINGS — доска находок по мандату «новые модели»

Файл ведётся ночными заходами по `MANDATE.md`: проверка интернета на модели, ещё не покрытые
атласом. Здесь только **находки и предложения** — правку глав атласа решает владелец.

Правило файла: утверждение без ссылки не пишется. Где вендор первоисточник не опубликовал —
так и помечено, а не додумано.

---

## Статус исполнения (обновлено 2026-07-28)

Владелец решил: находки по недостающим моделям внести в атлас. Внесено в **v1.5.0**.

| Находка | Решение | Куда легло |
|---|---|---|
| П1 · GPT-5.6 | ✅ внесено | `models/gpt.md § GPT-5.6`, матрица A–E, SKILL.md 2b/2c + блок обновлений + строка gap-анализа, `_universal.md`, README |
| П2 · Kimi K3 + K2.7-Code | ✅ внесено | `models/kimi.md` (две новые главы + переписано семейное правило о thinking), матрица A–E, `_universal.md`, антипаттерн #38 |
| П3 · Gemini 3.6 Flash / 3.5 Flash-Lite + депрекация sampling | ✅ внесено | `models/gemini.md` (две главы + правило #3 переписано), матрица A–E, кросс-вендорные таблицы `_universal.md`, Step 4 SKILL.md |
| П4 · Grok 4.5 | ✅ внесено | `models/grok.md § Grok 4.5` (в т.ч. регрессия окна 1M→500K и ценовая ступень 200K), матрица A–E, строка gap-анализа |
| П5 · Muse Spark 1.1 | ✅ внесено | **новый файл** `models/meta.md`, маршрут в SKILL.md Step 3, матрица A–E, README (10-е семейство) |
| П6 · Mistral Medium 3.5 | ✅ внесено | `models/mistral-frontier.md`, матрица A–E |
| П7 · мёртвые алиасы DeepSeek | ✅ поправлено | SKILL.md Step 2b — алиасы больше не «живой сигнал», а находка о миграции |
| П8 · граница Class 2 (MoE по активным параметрам) | ⏸ оставлено владельцу | переопределение рамки скилла, не механическая правка — решение вкусовое |
| П9 · GLM | — подтверждение, правки не требовалось | — |
| П10 · Qwen3.8-Max-Preview | ⏸ не заводим | лид без первоисточника; перепроверить следующим заходом |

**Что при исполнении перепроверялось и разошлось с этим файлом** (первоисточники читались заново
28.07, до правок):

1. **П2 — системный промпт K2.7-Code.** Здесь записано «вендор задаёт буквальный системный промпт».
   На карточке строка `You are Kimi, an AI assistant created by Moonshot AI.` стоит **в примере
   чата**, требованием не объявлена. В атлас пошла как иллюстративная; раздел про identity-pinning
   на её основании не переписывался.
2. **П3 — Gemini 3.5 Flash Cyber.** В официальном changelog за 21.07 его нет — только два GA-релиза
   (`gemini-3.6-flash`, `gemini-3.5-flash-lite`). Cyber есть в блог-посте, доступ закрытый; в атласе
   помечен как «существует, но не тюнится», отдельной строки в матрице не получил.

Остальное подтвердилось дословно, включая ключевые цитаты: цифры про худые промпты у GPT-5.6,
формулировку депрекации `temperature`/`top_p`/`top_k` в changelog Gemini, `always has thinking
enabled` у K3, `forces thinking and preserve_thinking as True` у K2.7-Code, рекомендацию Grok 4.5
и окна 500K/1M, карточку Mistral Medium 3.5, заявления Meta про 1M и самоуправление контекстом.
Уточнение по П1: миграционное правило вендора звучит как *"Preserve your current reasoning effort
as the baseline, then compare one level lower"* — смысл тот же, что записан, цитата в атласе точная.

---

## Заход 2026-07-28

Сверялось против фронта покрытия `SKILL.md` v1.4.0 (§ Step 2c). Claude не перепроверялся —
по нему сверка сделана 25.07 (Opus 5). Окно отставания — май–июль 2026.

**Итог: 7 находок + 1 фактическая устарелость + 1 лид без первоисточника.**
Отставание оказалось сильным: целое новое семейство OpenAI, новый фронтир Moonshot,
три новых Gemini и депрекация sampling-параметров, новый флагман xAI.

---

### 🔴 П1 — OpenAI GPT-5.6 (Sol / Terra / Luna). Самое сильное отставание

| | |
|---|---|
| **Модели** | GPT-5.6 Sol (флагман) · GPT-5.6 Terra · GPT-5.6 Luna |
| **API id** | `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna` |
| **Дата** | публично 9 июля 2026 |
| **Атлас знает** | до GPT-5.5 / 5.5 Instant (5 мая) — вся ветка 5.6 отсутствует |

**Первоисточники:**
- [developers.openai.com — Model guidance (latest model)](https://developers.openai.com/api/docs/guides/latest-model) — официальный гайд по семейству
- [developers.openai.com — модель `gpt-5.6-sol`](https://developers.openai.com/api/docs/models/gpt-5.6) — карточка: контекст 1 050 000, вход до 922 000, выход до 128 000, knowledge cutoff 16.02.2026

Дата публичного запуска — по [TechCrunch, 09.07.2026](https://techcrunch.com/2026/07/09/openai-launches-its-new-family-of-models-with-gpt-5-6/) и [Engadget](https://www.engadget.com/2210308/openai-rolls-out-gpt5-6-july-9/); на самой карточке OpenAI даты релиза нет.

**Чем промпт-поведение отличается от описанного в атласе** (всё — из официального гайда):

1. **Худые промпты выигрывают, и это измерено вендором.** «configurations with leaner system
   prompts improved evaluation scores by roughly 10–15% while reducing total tokens by 41–66%».
   Официальная рекомендация: убрать повторяющиеся инструкции, упростить описания инструментов,
   каждую инструкцию формулировать **один раз**.
   → Прямо бьёт по тому, что атлас советует для более ранних GPT-5.x; класс совета тот же,
   что инверсии у Opus 5 («over-prescriptive skills degrade output»), но у OpenAI это теперь
   с числом от вендора.
2. **Уровни reasoning effort расширены:** `none` · `low` · `medium` · `high` · **`xhigh`** · `max`.
   Старт рекомендован с `medium`, `max` — «для самых тяжёлых quality-first нагрузок».
   → Атлас описывает набор без `xhigh`.
3. **Миграционное правило вендора:** «Start with your current GPT-5.5 or GPT-5.4 reasoning
   setting, then test the same setting and one level lower» — то есть при переезде проверять
   уровень **ниже** текущего, а не тот же.
4. **Verbosity инвертировалась.** Модель по умолчанию лаконичнее GPT-5.5, поэтому старые
   «be brief / be concise» **пере**корректируют и делают ответ слишком коротким. Дефолт
   задавать параметром `text.verbosity`, а в промпте — только задачно-специфичное.
   → Ровно зеркало ситуации с Opus 5, где верно обратное (дефолт длинный, лаконичность нужно
   прописывать). Кросс-вендорная развилка, которой в атласе нет.
5. **Границы автономии просят прописывать явно:** «Define what level of action each request
   authorizes so the model can continue safe, in-scope work without unnecessary pauses».
6. **Тон — конкретикой, а не ярлыком:** вместо «friendly» описывать поведение
   («State the answer directly», когда признавать проблему).

---

### 🔴 П2 — Moonshot Kimi K3 (и Kimi K2.7-Code). Два пропущенных релиза подряд

| | |
|---|---|
| **Модели** | Kimi K3 · Kimi K2.7-Code |
| **id** | `moonshotai/Kimi-K3` · `moonshotai/Kimi-K2.7-Code` (API: `kimi-k2.7-code`) |
| **Дата** | K3 — 16 июля 2026 (API), открытые веса 26 июля · K2.7-Code — 12 июня 2026 |
| **Атлас знает** | K2.6 как текущий фронтир (апрель 2026) — пропущены оба |

**Первоисточники:**
- [huggingface.co/moonshotai/Kimi-K3](https://huggingface.co/moonshotai/Kimi-K3) — карточка модели
- [huggingface.co/moonshotai/Kimi-K2.7-Code](https://huggingface.co/moonshotai/Kimi-K2.7-Code) — карточка модели

Даты релизов на карточках явно не проставлены; 16.07 (API) / 26.07 (веса) — по
[qz.com](https://qz.com/moonshot-ai-kimi-k3-open-weights-download-072726), 12.06 для K2.7-Code — по
[MarkTechPost](https://www.marktechpost.com/2026/06/12/moonshot-ai-releases-kimi-k2-7-code-a-coding-model-reporting-21-8-on-kimi-code-bench-v2-over-k2-6/).

**Kimi K3 — что говорит карточка (промпт-релевантное):**
- 2.8T общих / 104B активных параметров, контекст 1 048 576, лицензия Kimi K3 License.
- **Thinking всегда включён** («Kimi K3 always has thinking enabled»), глубина — параметром
  reasoning effort `low` / `high` / `max`.
  → В атласе «thinking-on» для Kimi описан иначе; теперь у Moonshot это тоже out-of-band ручка,
  как у GLM-5.2 и всех прочих — паттерн «глубину рассуждения выносим наружу, а не вшиваем
  в текст» подтверждается ещё одним вендором.
- **Обязателен проброс `reasoning_content` обратно** в мультитёрне: полные assistant-сообщения
  с reasoning надо возвращать модели. → Усиливает строку матрицы про persistent reasoning.
- **Сэмплинг зависит от типа задачи:** top-p 0.95 для одношаговых, **top-p 1.0 для агентных**.
- Рекомендованный агентный каркас — Kimi Code CLI; движки vLLM / SGLang / TokenSpeed;
  API OpenAI- и Anthropic-совместимый на platform.kimi.ai.

**Kimi K2.7-Code — что говорит карточка:**
- 1T общих / 32B активных, контекст 256K, Modified MIT.
- **Thinking принудительный и отключить нельзя** — «forces thinking and preserve_thinking as True».
  → Это жёстче, чем всё, что атлас описывает про thinking-переключатели: промпты вида
  «не рассуждай / отвечай сразу» на этой модели структурно неисполнимы.
- **Вендор задаёт буквальный системный промпт:** `You are Kimi, an AI assistant created by
  Moonshot AI.` → Прямо задевает раздел атласа про identity-pinning.
- temperature 1.0 в thinking-режиме, top-p 0.95.
- Interleaved thinking + многошаговые tool-call'ы (та же схема, что K2 Thinking);
  расход thinking-токенов ~на 30% ниже K2.6.

---

### 🟠 П3 — Google: три новых Gemini + депрекация sampling-параметров

| | |
|---|---|
| **Модели** | `gemini-3.6-flash` (GA) · `gemini-3.5-flash-lite` (GA) · Gemini 3.5 Flash Cyber |
| **Дата** | 21 июля 2026 (все три) |
| **Атлас знает** | 3.1 Pro / 3.5 Flash / 3 Flash / 3.1 Flash-Lite — ветка 3.6 отсутствует |

**Первоисточники:**
- [ai.google.dev — Gemini API release notes](https://ai.google.dev/gemini-api/docs/changelog) — записи от 21.07.2026
- [blog.google — Introducing Gemini 3.6 Flash, 3.5 Flash-Lite, and 3.5 Flash Cyber](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/)

**Главное для атласа — не сами модели, а параметры:**

1. **`temperature`, `top_p`, `top_k` в Gemini API объявлены deprecated 21.07.2026**
   (запись в официальном changelog).
   → Это второй вендор подряд после Anthropic (у Claude sampling-параметры → HTTP 400 начиная
   с Opus 4.7, у Sonnet 5 тоже). Складывается **кросс-вендорный тренд**: тон и разнообразие
   больше не крутятся сэмплингом, только формулировкой. Атлас этот тренд пока описывает как
   claude-специфику.
2. **`gemini-3.6-flash`: −17% выходных токенов** против 3.5 Flash, до 65% экономии на DeepSWE;
   «higher precision with fewer unwanted code edits and reduced execution loops» (DeepSWE 49% vs 37%).
   → Класс совета тот же, что у GPT-5.6: модель сама стала лаконичнее, старые «пиши коротко»
   надо перепроверять.
3. **`gemini-3.5-flash-lite`** позиционируется вендором как **вариант под субагентов**
   (низкая задержка, 350 выходных токенов/с, SWE-Bench Pro 54.2% против 49.6% у 3 Flash).
   → Прямо релевантно главе про субагентов: у Google появился явный «дешёвый исполнитель».
4. **Gemini 3.5 Flash Cyber** — файнтюн 3.5 Flash под поиск и починку уязвимостей,
   доступ закрытый (правительства и доверенные партнёры через CodeMender).
   → Скорее пометка «существует, но недоступен», чем повод писать главу.

Промптингового гайданса Google к этим моделям не опубликовал — так и фиксирую, не додумываю.

Попутно (тот же changelog, июнь): `gemini-omni-flash-preview` (30.06), `gemini-3.1-flash-lite-image`
GA / Nano Banana 2 Lite (30.06) — мультимодальные, для промпт-атласа второстепенны.

---

### 🟠 П4 — xAI Grok 4.5

| | |
|---|---|
| **Модель** | Grok 4.5 |
| **Дата** | 8 июля 2026 |
| **Атлас знает** | «Grok 4.3 — текущий флагман; Grok 5 (цель Q2 2026) ещё не вышел — не тюнить под него» |

**Первоисточник:** [docs.x.ai — Models](https://docs.x.ai/docs/models) — Grok 4.5 числится
**рекомендованной** моделью: «the most intelligent and fastest model we've built»,
контекст **500K**, knowledge cutoff 01.02.2026, тарификация двумя ступенями (порог 200K токенов
промпта). Дата релиза на странице не проставлена; 08.07.2026 — по вторичным
([felloai](https://felloai.com/grok-4-5/), [Slipmp](https://www.slipmp.com/blog/grok-4-5-released-coding-agents-and-what-engineers-should-know/)).

**Дельта для атласа:**
- Строка «Grok 4.3 — current cost-efficient flagship» устарела: рекомендованная моделью вендора
  теперь 4.5.
- **Контекст УМЕНЬШИЛСЯ: 500K у 4.5 против 1M у 4.3** (обе цифры с той же страницы docs.x.ai).
  Это редкий случай регрессии окна при апгрейде — промпты, рассчитанные на 1M у Grok,
  при переезде на «более новую» модель ломаются. Такого предупреждения в атласе нет.
- Тарифная ступень на 200K означает, что раздутый системный промпт у Grok теперь имеет
  прямую ценовую границу.
- Явного промпт-гайданса и параметра reasoning effort в docs.x.ai для 4.5 не документировано.
- Утверждение атласа «Grok 5 не вышел» на 28.07.2026 всё ещё верно.

---

### 🟡 П5 — Meta Muse Spark 1.1 + публичный Meta Model API

| | |
|---|---|
| **Модель** | Muse Spark 1.1 |
| **Дата** | 9 июля 2026 |
| **Атлас знает** | Muse Spark (апрель 2026), «closed-weight, limited docs, most axes `?`» |

**Первоисточник:** [ai.meta.com — Introducing Muse Spark 1.1](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/)

**Дельта:** часть тех самых `?` закрывается официальным текстом —
- контекст 1M, мультимодальность (изображения, видео, PDF);
- заявлены **planning mode, goal conditioning, subagent delegation, context compaction**;
  модель «actively manage[s] its context window» — сама подтягивает раннее и уплотняет;
- открыт **Meta Model API** (public preview), **OpenAI-совместимый пакет**, structured output
  и параллельные tool-call'ы;
- режим «Thinking» в приложении Meta AI и на meta.ai.

То есть Meta впервые стала адресуемой из промпт-атласа как обычный API-вендор, а не как
«закрытая модель без документации». Отдельного prompting guide Meta не опубликовала.

---

### 🟡 П6 — Mistral Medium 3.5

| | |
|---|---|
| **Модель** | Mistral Medium 3.5 |
| **API id** | `mistral-medium-3-5-26-04` |
| **Дата** | 28 апреля 2026 (карточка); в ленте новостей Mistral фигурирует 22 мая 2026 |
| **Атлас знает** | Mistral Large 3 (дек 2025), Small 4 (мар 2026), Ministral 3-14B / 3-8B / 3-3B — среднего тира нет |

**Первоисточники:**
- [docs.mistral.ai — model card `mistral-medium-3-5-26-04`](https://docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04) — контекст 256K, открытые веса под modified MIT, $1.5/$7.5 за M токенов, «frontier-class multimodal model optimized for agentic and coding use cases», поддержка function calling / agents / structured outputs / FIM
- [mistral.ai/news](https://mistral.ai/news) — лента, где Medium 3.5 датирован 22.05.2026 («powering remote coding agents in Vibe»)

**Дельта промпт-поведения:** на карточке **нет** ни рекомендаций по системному промпту,
ни режима рассуждения, ни температуры. Так и фиксирую — писать главу не из чего, кроме
фактов доступности. Это и есть честный ответ по этой находке.

Прочее у Mistral за окно (для полноты, к промпт-атласу отношения не имеет):
Mistral OCR 4 (23.06), Leanstral 1.5 (02.07), Robostral Navigate (08.07, робототехника).

---

### 🟡 П7 — DeepSeek: новой модели нет, но атлас фактически устарел

**Первоисточник:** [api-docs.deepseek.com — Change Log](https://api-docs.deepseek.com/updates)

Последняя запись — 24 апреля 2026, релиз V4 (`deepseek-v4-pro`, `deepseek-v4-flash`).
Новых моделей в июне–июле в официальном changelog **нет** — это подтверждение, а не находка.

**Но там же:** «The two legacy API model names, `deepseek-chat` and `deepseek-reasoner`,
will be discontinued in three months (**2026-07-24**)».

→ Дата уже прошла (сегодня 28.07). При этом `SKILL.md` § Step 2b до сих пор перечисляет
`deepseek-chat` / `deepseek-reasoner` среди **сигналов распознавания вендора DeepSeek**,
а § Step 2c — среди актуальных опций. Это не новая модель, а **протухший факт в живом тексте**:
промпт, где встретился `deepseek-reasoner`, теперь указывает на мёртвый алиас, и совет
«это DeepSeek, тюним под V3.2/R1» будет неверным.

Отдельно: о выходе V4 из preview в GA в середине июля с peak-hour ценообразованием пишет
[TechNode (30.06.2026)](https://technode.com/2026/06/30/deepseek-to-launch-v4-in-mid-july-with-new-peak-time-api-pricing/), но
**в официальном changelog этого нет** — как подтверждённое не засчитываю.

---

### ⚪ П8 — Локальные малые (Class 2, 2-9B): отставания не найдено

Проверено 28.07.2026 по каждому семейству из наряда:

| Семейство | Что нашлось | Вердикт |
|---|---|---|
| **Gemma** | [Gemma 4, 02.04.2026](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/): E2B, E4B, 26B MoE, 31B Dense, Apache 2.0, 128K (edge) / 256K | E2B/E4B **уже покрыты** матрицей; 26B/31B — выше диапазона 2-9B |
| **Qwen small** | Открытые веса 3.6 — [Qwen3.6-27B](https://huggingface.co/Qwen/Qwen3.6-27B) (апр 2026, Apache 2.0) и [Qwen3.6-35B-A3B](https://huggingface.co/Qwen/Qwen3.6-35B-A3B) | Обе **выше 9B**; вариантов 2-9B в линейке 3.6 не выпущено (в 3.5 они есть и покрыты) |
| **Phi** | Последнее — Phi-4-reasoning-vision-15B (март 2026) | 15B, **выше диапазона**; Phi-4-mini в матрице актуален |
| **Llama** | Открытых релизов после Llama 4 (апр 2025) нет; Meta ушла в закрытый Muse Spark | Llama 3.2 в матрице по-прежнему актуальна |
| **Ministral** | Новых Ministral за май–июль нет (Mistral выпускал Medium 3.5, OCR 4, Leanstral, Robostral) | Ministral 3B в матрице актуален |

**Вывод: по Class 2 атлас не отстал.** Ни одна модель 2-9B, вышедшая после текущего покрытия,
не найдена.

Но всплыл **вопрос о границе класса**, а не о покрытии: `Qwen3.6-35B-A3B` (3B активных) и
`Gemma 4 26B-A4B` (4B активных) — это MoE, которые по активным параметрам и по требованиям
к железу ведут себя как локальные малые, а формулировка класса «2-9B» отсекает их по общему
числу параметров. Это развилка определения, решать владельцу (см. предложения ниже).

---

### ⚪ П9 — Z.ai GLM: нового нет

**Первоисточник:** [docs.z.ai — New Released](https://docs.z.ai/release-notes/new-released).
GLM-5.2 (июнь 2026) остаётся флагманом; в июле Z.ai выпустил **ZCode** — harness/агентный
каркас поверх GLM-5.2, не модель ([SCMP](https://www.scmp.com/tech/tech-trends/article/3359170/zhipu-ai-releases-harness-glm-52-model-chinese-firm-takes-aim-anthropic)).
Покрытие атласа по GLM актуально.

---

### ⚫ П10 — Qwen3.8-Max-Preview: лид БЕЗ первоисточника, добавлять пока не из чего

Заявлен 19.07.2026 на WAIC в Шанхае; hosted id называют `qwen3.8-max-preview`, доступ —
Alibaba Cloud Token Plan / Qoder / QoderWork.

**Проблема:** первоисточника нет. Проверено 28.07.2026:
- в [Alibaba Cloud Model Studio — Models](https://www.alibabacloud.com/help/en/model-studio/models)
  модель **не числится**: последнее поколение в списке — `qwen3.7-max`, `qwen3.7-plus`, `qwen3.6-flash`;
- на [huggingface.co/Qwen](https://huggingface.co/Qwen) семейства 3.8 нет;
- официального блог-поста Qwen о запуске не найдено (qwenlm.github.io отдаёт архив до сентября 2025,
  qwen.ai/blog не отдаёт содержимого).

Вторичные источники ([Bloomberg](https://www.bloomberg.com/news/articles/2026-07-19/alibaba-s-qwen-unveils-preview-of-flagship-ai-model),
[SCMP](https://www.scmp.com/tech/article/3361119/alibaba-says-newest-qwen-ai-model-second-only-anthropics-claude-fable-5),
[MarkTechPost](https://www.marktechpost.com/2026/07/19/alibaba-previews-qwen3-8-max-a-2-4-trillion-parameter-multimodal-model-days-after-moonshots-kimi-k3-open-weight-launch/))
сходятся в одном: Alibaba не опубликовала ни карточку модели, ни бенчмарк-таблицу, ни цену,
ни лицензию — только заявление со сцены.

**Промпт-релевантной информации нет вовсе.** Писать в атлас нечего.

---

## Предложения владельцу (решение — за ним)

Приоритет сверху вниз, тем же порядком, что находки.

1. **GPT-5.6** — предложение: раздел в `models/gpt.md` + новая строка «Recent updates» в `SKILL.md`
   § Step 2c. Отдельно стоит вынести **кросс-вендорную инверсию verbosity**: у GPT-5.6 старое
   «be brief» вредит (модель уже лаконична), у Opus 5 — наоборот, лаконичность надо прописывать.
   Сейчас в атласе есть только claude-половина этой развилки.
2. **Kimi K3 + K2.7-Code** — предложение: обновить `models/kimi.md`; K2.7-Code интересен не как
   ещё одна модель, а как **первый в атласе случай нескрываемого thinking** (`forces thinking`)
   и как вендор, диктующий дословный системный промпт.
3. **Gemini 3.6 Flash / 3.5 Flash-Lite** — предложение: обновить `models/gemini.md`; главное —
   **депрекация `temperature`/`top_p`/`top_k`**: похоже, пора поднимать её из claude-специфики
   в кросс-вендорное правило (`_universal.md`), а не описывать вторым частным случаем.
   Плюс `3.5 Flash-Lite` как штатный «дешёвый субагент» в главу про субагентов.
4. **Grok 4.5** — предложение: обновить `models/grok.md`; отдельно предупреждение о **сокращении
   контекста 1M → 500K при апгрейде с 4.3** — редкий случай, когда «новее» значит «меньше окно».
5. **DeepSeek** — предложение: это не добавление, а **правка протухшего**: убрать
   `deepseek-chat` / `deepseek-reasoner` из живых сигналов Step 2b (алиасы выведены 24.07.2026),
   оставив их как исторические. Дешёвая правка, ошибочный совет сегодня даёт прямо сейчас.
6. **Muse Spark 1.1** — предложение: поднять Meta с «most axes `?`» до нормальной короткой главы:
   Meta Model API, OpenAI-совместимость, 1M контекст, subagent delegation и context compaction —
   этого хватает на осмысленный раздел.
7. **Mistral Medium 3.5** — предложение: добавить строкой в перечень версий Mistral.
   Полноценную главу писать не из чего — вендор промпт-гайданса не дал; так и стоит пометить
   в тексте, чтобы следующий заход не искал заново.
8. **Граница Class 2** — предложение-развилка, а не задача: расширить определение малых локальных
   с «2-9B» до «≤~9B активных параметров», чтобы MoE вроде `Qwen3.6-35B-A3B` и `Gemma 4 26B-A4B`
   попадали в класс по тому признаку, который реально определяет и железо, и поведение.
   Решение вкусовое — это переопределение рамки скилла, не механическая правка.
9. **Qwen3.8-Max-Preview** — предложение: **не заводить** до появления карточки модели или
   официального поста. Держать в этом файле как лид и перепроверить следующим заходом.
10. **Процедурное предложение по самому мандату:** заход показал, что за ~2 месяца без сверки
    накопилось 7 находок, две из которых (GPT-5.6, Kimi K3) — целые пропущенные фронтир-семейства.
    Если мандат правда ночной, стоит гонять его по расписанию, а не вручную; и стоит решить,
    добавляется ли найденное автоматически или всегда через этот файл (сейчас — через файл).
