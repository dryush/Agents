# Agents — система ИИ-агентов для разработки

Репозиторий содержит **готовую к использованию инструкцию-систему** для организации разработки проекта командой ИИ-агентов.
Система описывает: координацию через `AGENTS.md`, хранение знаний через Memory Bank, набор специализированных субагентов, навыки (skills) и инженерные стандарты.

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

## Концептуальная схема

```
Пользователь
    │
    ▼
AGENTS.md / .qwen/  ←── точки входа для LLM-клиентов
    │
    ▼
Координатор (AGENTS.md)
    │  делегирует задачи по TASK-ID
    ▼
Субагенты (.qwen/agents/)
    │  читают и пишут через
    ▼
Memory Bank (.memory-bank/)  ←── единственный источник истины
    │  через skill
    ▼
memory-bank-owner (.qwen/skills/memory-bank-owner/)
```

Субагенты **автономны**: читают контекст и записывают артефакты самостоятельно через `memory-bank-owner`, не возвращая содержимое координатору.

---

## Структура репозитория

```
.
├── AGENTS.md                          # Точка входа для Gemini CLI и любого LLM
├── README.md                          # Этот файл
│
├── .qwen/                             # Конфигурация QwenCode CLI
│   ├── agents/                        # Субагенты: инструкции + инструменты
│   │   ├── architect.md               # Проектирует FSD-архитектуру, application layer, ADR
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
│   └── skills/                        # Переиспользуемые навыки (вызываются субагентами)
│       ├── memory-bank-owner/         # Операции над .memory-bank/: READ, WRITE, UPDATE-STATUS
│       │   └── SKILL.md               # Полный протокол всех операций
│       ├── branch-manager/            # Создание и переключение веток по TASK-ID
│       │   └── SKILL.md
│       ├── commit-manager/            # Формирование conventional commits
│       │   └── SKILL.md
│       └── system-env/                # Обнаружение и кэширование окружения (OS, Node, Python…)
│           └── SKILL.md
│
└── .memory-bank/                      # Единственный источник истины (SSoT) проекта
    ├── index.md                       # Навигационный индекс — читать ПЕРВЫМ
    ├── system-env.md                  # Кэш окружения (ОС, рантаймы, пакетные менеджеры)
    │
    ├── mbb/
    │   └── principles.md              # Правила Memory Bank: кто читает, кто пишет, структура
    │
    ├── spec/
    │   ├── engineering/
    │   │   ├── guardrails.md          # Архитектура (FSD + Application Layer), KISS, logging ★
    │   │   ├── quality-gate.md        # Порядок и критерии прохождения CI-шагов
    │   │   ├── testing-standards.md   # TDD-цикл, матрица покрытия, уровни тестов
    │   │   └── code-comments.md       # Стандарт комментирования кода
    │   └── process/
    │       ├── pipeline.md            # Конвейер фаз: PLAN→SPEC→ARCH→TDD→DEV→QA→RELEASE
    │       └── status-protocol.md     # Протокол статусов задач и переходов между фазами
    │
    ├── _common/                       # Универсальные tech-guides (накапливаются агентами)
    │   ├── index.md                   # Индекс существующих guides
    │   └── README.md                  # Правила создания нового guide
    │
    └── tasks/                         # Артефакты конкретных задач (создаются в процессе работы)
        └── <TASK-ID>/                 # FT-XXXX | BG-XXXX | TD-XXXX | EP-XXX
            ├── plan.md                # Цель, скоуп, декомпозиция
            ├── spec.md                # Acceptance criteria + CLI Contract
            ├── architecture.md        # FSD-структура, контракты, logging-таблица
            ├── dev-guide.md           # Команды, стек, переменные окружения
            ├── status.md             # Текущая фаза и статус задачи
            └── adr-N.md              # Architectural Decision Records (при необходимости)
```

> ★ — файл с наибольшим влиянием на все решения агентов.

---

## Навигация для LLM: какие файлы читать

### Перед любой правкой
1. `.memory-bank/index.md` — навигационный индекс, всегда первый
2. `.memory-bank/mbb/principles.md` — правила работы с Memory Bank
3. `.memory-bank/spec/engineering/guardrails.md` — архитектурные запреты и стандарты

### Изменить поведение агента
→ `.qwen/agents/<agent-name>.md`
Каждый файл содержит: `<role>`, `<context>` (что читать), `<grounding>` (как читать через skill), `<rules>`, `<output>`, `<output_contract>`.

### Изменить навык (skill)
→ `.qwen/skills/<skill-name>/SKILL.md`
Все операции задокументированы внутри файла с примерами вызова.

### Изменить процесс разработки (фазы, порядок агентов)
→ `.memory-bank/spec/process/pipeline.md`
→ `.memory-bank/spec/process/status-protocol.md`

### Изменить инженерные стандарты
| Что менять | Файл |
|------------|------|
| Архитектура (слои, зависимости, logging) | `.memory-bank/spec/engineering/guardrails.md` |
| Критерии CI (lint, typecheck, tests, build) | `.memory-bank/spec/engineering/quality-gate.md` |
| TDD-цикл, матрица покрытия, структура тестов | `.memory-bank/spec/engineering/testing-standards.md` |
| Стандарт комментариев | `.memory-bank/spec/engineering/code-comments.md` |

### Изменить точку входа координатора
→ `AGENTS.md` — содержит инструкции для Gemini CLI и общую логику координации.
Координатор **не читает артефакты задач** и **не пишет в Memory Bank** — это ответственность субагентов.

### Добавить tech-guide (паттерн, антипаттерн, особенность технологии)
→ `.memory-bank/_common/` — создать новый файл через skill `memory-bank-owner` (операция `UPSERT-KNOWLEDGE`).
Формат: `Problem / Solution / Example / Enforcement`.

---

## Ключевые принципы

**Memory Bank как SSoT.** Любое знание, не зафиксированное в `.memory-bank/`, не существует для агентов. Субагенты читают и пишут исключительно через skill `memory-bank-owner`.

**Автономность субагентов.** Координатор делегирует задачу по `TASK-ID`. Субагент самостоятельно читает контекст, выполняет работу, записывает артефакты, обновляет `status.md` и возвращает координатору только статус.

**Feature-Slice Design + Application Layer.** Архитектура кода строится по FSD. `application/` описывает все действия системы. Критерий готовности: каждый use-case вызываем из неинтерактивного CLI без рефакторинга.

**Структурное логирование и трассировка.** Каждая операция порождает JSON-лог с `traceId`, `spanId`, `module`, `durationMs`. `traceId` передаётся явным параметром через все слои. Подробная спецификация — в `guardrails.md`, блок `<logging_tracing>`.

**Иммутабельные файлы.** Следующие файлы не изменяются агентами без ADR и явного решения пользователя:
- `.memory-bank/mbb/principles.md`
- `.memory-bank/spec/process/pipeline.md`
- `.memory-bank/spec/process/status-protocol.md`
- `.memory-bank/spec/engineering/guardrails.md`
