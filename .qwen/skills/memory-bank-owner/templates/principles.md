---
type: principles
version: 1.0.0
---
# Memory Bank Principles

<sst>
Memory Bank — единственный источник истины о состоянии проекта.
Любое знание, не зафиксированное в memory-bank, не существует для агентов.
</sst>

<write_rules>
- Все записи выполняются строго через skill memory-bank-owner.
- Ни один агент не пишет в .memory-bank/ напрямую.
- Субагент самостоятельно записывает артефакты через memory-bank-owner (WRITE-ARTIFACT).
- Субагент самостоятельно обновляет status.md через memory-bank-owner (UPDATE-STATUS).
- Координатор НЕ пишет в memory-bank — это исключительная ответственность субагентов.
</write_rules>

<read_rules>
- Субагенты читают файлы через memory-bank-owner (READ-CONTEXT), не напрямую.
- Координатор не читает артефакты задачи — только субагенты.
- Никогда не используй знание из прошлых сессий без повторного чтения.
</read_rules>

<structure>
Обязательная структура директорий:

  .memory-bank/
  ├── index.md                        ← навигационный индекс
  ├── mbb/
  │   └── principles.md               ← правила memory-bank
  ├── system-env.md                   ← кеш окружения
  ├── adr/                            ← глобальные ADR проекта
  │   └── index.md
  ├── _common/                        ← универсальные tech-guides
  │   ├── index.md
  │   └── README.md
  ├── spec/
  │   ├── process/
  │   │   ├── pipeline.md
  │   │   └── status-protocol.md
  │   ├── engineering/
  │   │   ├── guardrails.md
  │   │   ├── quality-gate.md
  │   │   ├── testing-standards.md
  │   │   └── code-comments.md
  │   └── agents/
  │       └── <agent-name>.md
  └── tasks/
      └── <TASK-ID>/
          ├── plan.md
          ├── status.md
          ├── spec.md
          ├── architecture.md
          ├── dev-guide.md
          └── adr-N.md                ← task-level ADR (при необходимости)
</structure>

<naming>
TASK-ID формат:   FT-XXXX (feature), BG-XXXX (bug), TD-XXXX (techdebt), EP-XXX (epic)
Файлы артефактов: kebab-case.md
Директории:       kebab-case
Global ADR:       .memory-bank/adr/adr-NNN-<slug>.md
Task ADR:         .memory-bank/tasks/<TASK-ID>/adr-N.md
Tech-guides:      .memory-bank/_common/<technology>/guide.md
</naming>

<referential_integrity>
- Каждый файл в tasks/ должен быть зарегистрирован в index.md.
- Каждый глобальный ADR регистрируется в .memory-bank/adr/index.md.
- При переименовании или удалении файла — обновить все ссылки на него.
- Ссылки всегда относительные от корня репозитория.
- Мёртвые ссылки недопустимы — проверяются при каждой операции WRITE-ARTIFACT.
</referential_integrity>

<immutability>
Следующие файлы не изменяются агентами без явного решения пользователя:
  - .memory-bank/mbb/principles.md
  - .memory-bank/spec/process/pipeline.md
  - .memory-bank/spec/process/status-protocol.md
  - .memory-bank/spec/engineering/guardrails.md
Для изменения этих файлов требуется ADR.
</immutability>

<evolution>
Агенты ДОЛЖНЫ создавать инструкции в _common/ при обнаружении:
  - нового эффективного паттерна решения
  - антипаттерна или причины сбоя
  - технологической особенности, требующей специфичной настройки
  - неоднозначности в текущих правилах

Формат инструкции: Problem / Solution / Example / Enforcement
Операция: UPSERT-KNOWLEDGE через memory-bank-owner.
</evolution>
