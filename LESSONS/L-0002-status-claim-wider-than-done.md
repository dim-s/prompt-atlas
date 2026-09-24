---
id: L-0002
addressee: executor
trigger: [changelog, briefing, status-doc, verified-claim, cross-checked, second-source, pushed, done-claim]
target: "статусный документ (CHANGELOG, BRIEFING) пишет сделанное/проверенное одной общей фразой шире, чем реально сделано"
status: candidate
evidence:
  hits: 2
  situations:
    - { id: s-v170, date: 2026-08-29, where: "CHANGELOG [1.7.0] / BRIEFING", what: "пуш описан свершённым в прошедшем времени до пуша (дерево грязное, main ahead 1)", caught-by: dmitry-manager, ref: "30c845f; бывш. QUARANTINE п.1" }
    - { id: s-5e2336-c, date: 2026-09-24, where: "CHANGELOG [1.9.0] шапка", what: "«numeric claims cross-checked against a second page» — для Qwen Omni-Flash и GPT-6 Sol/Luna второй страницы не было", caught-by: dmitry-manager (гейт р.1), ref: "s-5e2336; v1.9.0 working tree" }
  helped: null
  refuted: 0
born: 2026-09-24
last-touched: 2026-09-24
---
Правило: вместо общей фразы о сделанном («проверено по второму источнику», «запушено») пиши утверждение поштучно — к каждому охваченному пункту указатель-улику (URL второй страницы, хэш коммита); пункт без указателя явно выноси в «не проверено / ещё не сделано», а несвершённое пиши в плановом залоге и меняй на факт с хэшем после свершения.
Почему: общая формулировка по умолчанию покрывает все пункты, а делалось не для всех — читатель статус-дока не видит исключений. Указатель на пункт снимает суждение «кажется, проверил всё»: где нечего подставить, там и не сделано.
