# Agents — библиотека ИИ-агентов для разработки

Этот репозиторий **является самой агентной системой**, а не проектом, который она обслуживает.
Здесь хранятся инструкции агентов, навыки (skills), инженерные стандарты и контекст (Memory Bank).

Если ты — ИИ-агент, работающий с этим репозиторием, твоя задача:
**дорабатывать, улучшать и расширять систему**, а не выполнять задачи разработки по её правилам.

Совместима с **QwenCode CLI** (`.qwen/`) и **Gemini CLI** (`AGENTS.md`).

---

## Документация QwenCode CLI

| Раздел | Ссылка |
|--------|--------|
| Skills (навыки) | [qwen-code/skills](https://qwen-coder.readthedocs.io/en/latest/skills.html) |
| Subagents (субагенты) | [qwen-code/subagents](https://qwen-coder.readthedocs.io/en/latest/subagents.html) |
| Settings (настройки) | [qwen-code/settings](https://qwen-coder.readthedocs.io/en/latest/settings.html) |
| MCP (инструменты) | [qwen-code/mcp](https://qwen-coder.readthedocs.io/en/latest/mcp.html) |

---

## Что здесь хранится

```
.
├── AGENTS.md                          # Координатор: точка входа Gemini CLI и любого LLM
├── README.md                          # Этот файл
│
├── .qwen/                             # Конфигурация QwenCode CLI
│   ├── agents/                        # Субагенты — каждый файл: один специализированный агент
│   │   ├── architect.md               # Проектирует FSD-архитектуру, Application Layer, ADR
│   │   ├── code-reviewer.md           # Проверяет guardrails, типизацию, logging
│   │   ├── developer.md               # Реализует код (RED→GREEN, не трогает тесты)
│   │   ├── docs-writer.md             # Создаёт и обновляет документацию
│   │   ├── integration-test-writer.md # Пишет integration и e2e тесты
│   │   ├── planner.md                 # Декомпозирует запрос, формирует plan.md
│   │   ├── qa-validator.md            # Аудит покрытия, CLI-contract, logging-checklist
│   │   ├── release-manager.md         # Финальный чеклист, единственный с правом merge
│   │   ├── spec-writer.md             # Формирует acceptance criteria + CLI Contract
│   │   ├── tech-lead.md               # Создаёт/обновляет dev-guide
│   │   └── unit-test-writer.md        # Пишет unit-тесты до реализации (фаза RED)
│   │
│   └── skills/                        # Переиспользуемые навыки — вызываются субагентами
│       ├── memory-bank-owner/         # READ / WRITE / UPDATE-STATUS для .memory-bank/
│       │   └── SKILL.md               # Полный протокол всех операций
│       ├── branch-manager/            # Создание и переключение веток по TASK-ID
│       │   └── SKILL.md
│       ├── commit-manager/            # Conventional commits
│       │   └── SKILL.md
│       └── system-env/                # Обнаружение и кэширование окружения
│           └── SKILL.md
│
└── .memory-bank/                      # Единственный источник истины (SSoT) о системе
    ├── index.md                       # Навигационный индекс — читать ПЕРВЫМ
    ├── system-env.md                  # Кэш окружения (ОС, рантаймы, пакетные менеджеры)
    ├── mbb/
    │   └── principles.md              # Правила Memory Bank: кто читает, кто пишет
    ├── spec/
    │   ├── engineering/
    │   │   ├── guardrails.md          # ★ Архитектурные инварианты, logging, KISS
    │   │   ├── quality-gate.md        # Критерии CI: lint, typecheck, tests, build
    │   │   ├── testing-standards.md   # TDD-цикл, матрица покрытия, уровни тестов
    │   │   └── code-comments.md       # Стандарт комментирования
    │   └── process/
    │       ├── pipeline.md            # Конвейер: PLAN→SPEC→ARCH→TDD→DEV→QA→RELEASE
    │       └── status-protocol.md     # Протокол статусов и переходов между фазами
    ├── _common/                       # Tech-guides — накапливаются агентами
    │   ├── index.md                   # Индекс существующих guides
    │   └── README.md                  # Правила создания нового guide
    └── tasks/                         # Артефакты задач (FT-XXXX, BG-XXXX, TD-XXXX)
        └── <TASK-ID>/
            ├── plan.md
            ├── spec.md
            ├── architecture.md
            ├── dev-guide.md
            ├── status.md
            └── adr-N.md
```

> ★ — файл с наибольшим влиянием на все решения агентов.

---

## Анатомия субагента `.qwen/agents/<name>.md`

Каждый файл субагента — это самодостаточная инструкция. Стандартная структура:

```
frontmatter (name, version, description)
<role>         — одна роль, одна ответственность
<context>      — какие файлы читать перед работой (через skill memory-bank-owner)
<grounding>    — как вызывать skill memory-bank-owner для загрузки контекста
<rules>        — что делать / что запрещено
<output>       — что производить
<output_contract> — формат, статус возврата (DONE / BLOCKED / QUESTION)
```

Правила для агента, дорабатывающего субагент:
- Роль формулируется одним глаголом + существительным («Проектирует», «Реализует», «Проверяет»).
- Каждый субагент читает контекст **только через** skill `memory-bank-owner`; прямое чтение файлов запрещено.
- Субагент **возвращает статус** координатору, но **не возвращает содержимое артефактов** — он записывает их сам через `memory-bank-owner`.
- Запрещено добавлять субагенту обязанности другой роли (нарушение Single Responsibility).

---

## Анатомия навыка `.qwen/skills/<name>/SKILL.md`

Skill — повторно используемый протокол с явным списком операций. Стандартная структура:

```
frontmatter (name, version)
<description>     — назначение и область применения
<operations>      — список именованных операций с параметрами
<operation name="OP-NAME">
    <input>       — обязательные и опциональные параметры
    <steps>       — последовательность действий
    <output>      — формат возврата
    <errors>      — коды ошибок и действия
</operation>
<caller_identity> — правило: вызывающий передаёт agent_name явным параметром
```

Правила для агента, дорабатывающего skill:
- Каждая операция атомарна: один вызов — одно действие.
- Новая операция добавляется в `<operations>`, не изменяя существующие (обратная совместимость).
- `caller_identity` — обязательный параметр всех операций; skill отклоняет запросы без него.
- После добавления операции — обновить `index.md` в `.memory-bank/_common/` если skill публичный.

---

## Как дорабатывать систему: навигация по задачам

### Добавить нового субагента
1. Прочитай `.memory-bank/spec/engineering/guardrails.md` — архитектурные ограничения.
2. Прочитай `.memory-bank/spec/process/pipeline.md` — где агент встаёт в конвейер.
3. Прочитай `.qwen/agents/developer.md` или похожий агент как образец структуры.
4. Создай `.qwen/agents/<name>.md` по структуре из раздела «Анатомия субагента».
5. Добавь агента в `AGENTS.md` (секция делегации координатора).
6. Добавь quality gate для нового агента в `.memory-bank/spec/process/pipeline.md`.

### Изменить поведение существующего субагента
1. Прочитай целевой файл `.qwen/agents/<name>.md` — понять текущий контракт.
2. Прочитай `.memory-bank/spec/process/pipeline.md` — не нарушает ли правка порядок фаз.
3. Если правка меняет `<output_contract>` — также обновить `AGENTS.md` (таблица ожидаемых статусов).

### Добавить или расширить навык (skill)
1. Прочитай `.qwen/skills/<name>/SKILL.md` — текущий протокол.
2. Добавь новую `<operation>` в конец блока `<operations>`.
3. Не меняй сигнатуру или поведение существующих операций.
4. Обнови `index.md` в `.memory-bank/_common/` при изменении публичного интерфейса.

### Изменить процесс (фазы, порядок агентов, quality gates)
1. Прочитай `.memory-bank/spec/process/pipeline.md` — полный конвейер.
2. Прочитай `.memory-bank/spec/process/status-protocol.md` — переходы статусов.
3. Любое изменение порядка фаз или добавление обязательного этапа — создай ADR:
   `.memory-bank/tasks/<TASK-ID>/adr-1.md` с разделами `Context / Decision / Consequences`.
4. Обнови `AGENTS.md` (раздел «Стандартный поток»).

### Изменить инженерные стандарты
| Что менять | Файл |
|------------|------|
| Архитектурные слои, зависимости, logging | `.memory-bank/spec/engineering/guardrails.md` |
| Критерии CI (lint, typecheck, tests, build) | `.memory-bank/spec/engineering/quality-gate.md` |
| TDD-цикл, матрица покрытия, структура тестов | `.memory-bank/spec/engineering/testing-standards.md` |
| Стандарт комментариев в коде | `.memory-bank/spec/engineering/code-comments.md` |

> Все четыре файла иммутабельны без явного ADR — изменять только вместе с созданием `adr-N.md`.

### Добавить tech-guide (паттерн, антипаттерн, особенность технологии)
1. Прочитай `.memory-bank/_common/README.md` — формат `Problem / Solution / Example / Enforcement`.
2. Создай `.memory-bank/_common/<topic>.md`.
3. Добавь запись в `.memory-bank/_common/index.md`.

### Изменить координатор
→ `AGENTS.md` — логика делегирования, классификация задач, quality gates, таблица статусов.
Помни: координатор **не читает артефакты задач** и **не пишет в Memory Bank** напрямую — это ответственность субагентов.

---

## Ключевые инварианты системы

Эти правила нельзя нарушать при доработке системы:

**Memory Bank — единственный SSoT.** Любое знание, не зафиксированное в `.memory-bank/`, не существует для агентов. Субагенты читают и пишут только через skill `memory-bank-owner`.

**Один агент — одна роль.** Субагент выполняет ровно одну фазу конвейера. Расширение роли = добавление нового субагента.

**Автономность субагента.** Субагент сам читает контекст, выполняет работу, пишет артефакты, обновляет `status.md`. Возвращает координатору только статус (`DONE` / `BLOCKED` / `QUESTION`).

**Обратная совместимость skills.** Существующие операции skills не меняются — только расширяются новыми операциями.

**caller_identity обязателен.** Каждый вызов skill содержит явный `agent_name`. Skills отклоняют анонимные запросы.

**Иммутабельные файлы** (требуют ADR для изменения):
- `.memory-bank/mbb/principles.md`
- `.memory-bank/spec/process/pipeline.md`
- `.memory-bank/spec/process/status-protocol.md`
- `.memory-bank/spec/engineering/guardrails.md`
