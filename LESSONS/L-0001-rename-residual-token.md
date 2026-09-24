---
id: L-0001
addressee: executor
trigger: [rename, model-version, new-model, cross-vendor-claim, residual-token, stale-claim, sampling, grep, follow-up, every-other-current, first-documented, only-model, quantifier-claim]
target: "утверждение о множестве моделей (старое, ставшее ложным от изменения, или своё новое «first/only/every») не сверено grep-ом с репозиторием — правят по перечню известных мест, а не по классу"
status: active
evidence:
  hits: 7
  situations:
    - { id: s-1, date: 2026-08-29, where: "references/matrix.md:86", what: "устаревшее «Muse Spark 1.1» осталось в сноске", caught-by: dmitry-manager, ref: "references/matrix.md:86" }
    - { id: s-2, date: 2026-08-29, where: "references/matrix.md:71/131/188/232/251", what: "токен остался в строках Meta A–E после первого исправления", caught-by: dmitry-manager, ref: "references/matrix.md:71,131,188,232,251" }
    - { id: s-3, date: 2026-08-29, where: "SKILL.md:388/494", what: "токен остался в harness-таблице и маршрутизации после второго исправления", caught-by: dmitry-manager, ref: "SKILL.md:388,494" }
    - { id: s-4ede53-a, date: 2026-09-13, where: "references/models/_universal.md:343/408-410/492", what: "GPT-6 Astra без sampling делает ложным «only GPT-5.x and older Claude still tune»; план отложил правку в follow-up", caught-by: dmitry-manager (гейт ПЛАН р.1), ref: "план v1.8.0; f314293" }
    - { id: s-4ede53-b, date: 2026-09-13, where: "agentic-systems/gemini-cli.md:228", what: "ячейка «Codex CLI: Tunable» пропущена по непроверенному «легаси-таблица 2.5»; второй раунд подряд новое место класса", caught-by: dmitry-manager (гейт ПЛАН р.2), ref: "план v1.8.0; f314293" }
    - { id: s-5e2336-a, date: 2026-09-24, where: "references/techniques.md:416", what: "«every other current Claude» молчало про Opus 5.5; grep по свойствам (статус, off-switch, sampling) квантор по семейству не ловит", caught-by: dmitry-manager (гейт р.1), ref: "s-5e2336; v1.9.0 working tree" }
    - { id: s-5e2336-b, date: 2026-09-24, where: "references/models/qwen-frontier.md:168", what: "своё новое «first documented Qwen knob» ложно — у Max уже `reasoning_effort` (qwen-frontier.md:43)", caught-by: dmitry-manager (гейт р.1), ref: "s-5e2336; v1.9.0 working tree" }
  helped: true
  refuted: 0
born: 2026-08-29
last-touched: 2026-09-24
---
Правило: когда изменение делает факт о модели иным — переименование, новая модель, снятый/добавленный параметр — сначала сформулируй затронутое УТВЕРЖДЕНИЕ как класс (старое имя; «кто ещё тюнит sampling»; ступень effort-ладдера) и найди его grep-ом по ВСЕМУ репозиторию, а не по местам, которые помнишь или указал ревьюер. К grep по свойству добавь grep по КВАНТОРАМ над семейством новой модели (`every|all|only|other|rest|first|any` рядом с именем вендора/семейства или `current`): фраза «every other current Claude» не содержит ни имени новой модели, ни токена свойства, и grep по свойствам её не видит. То же — до того, как пишешь СВОЁ утверждение с квантором («first documented X knob», «only Y supports», «inverts Z»): сперва grep этого свойства по файлу вендора и репо, иначе новая строка ложна против уже записанного. Правь каждое живое вхождение в том же изменении, не выноси в follow-up: CONTRIBUTING §«How to update an existing cell» требует чинить ячейку, ставшую неверной. Вхождение пропускаешь — сперва открой саму строку и назови, почему она не того класса; обоснование по памяти о файле не годится. Упоминания в истории прошлых версий (CHANGELOG, записи «когда X стал доступен», титулы первоисточников, словари сигнальных токенов) законны и не правятся.
Почему: точечный поиск структурно не покрывает дерево — в двух сессиях по два-три раунда гейта подряд находилось новое место того же класса, потому что правили перечень мест, а не класс. Сработало на v1.8.0 р.3: grep по классу (sampling + `none` в effort-ладдере) и правило вместо перечня → финальный гейт чистый. v1.9.0 р.1: grep по свойствам исполнитель сделал сам, но кванторные обобщения («every other current», «first documented») мимо него — отсюда шаг с кванторами. Различение «живое/история» нужно, чтобы grep по репо не стал чисткой законной истории.
