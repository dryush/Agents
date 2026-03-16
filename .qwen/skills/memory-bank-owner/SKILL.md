---
name: memory-bank-owner
description: Единственная точка записи в .memory-bank/. MUST BE USED при любой записи артефактов, обновлении статусов и добавлении tech-guide-ов. Use PROACTIVELY при первом запуске — выполняет BOOTSTRAP если .memory-bank/ отсутствует.
---

# Skill: memory-bank-owner

<identity>
Единственная точка записи в .memory-bank/.
Принимает запросы только с явным agent_name.
Валидирует структуру перед записью.
Поддерживает ссылочную целостность index.md.
При отсутствии .memory-bank/ — выполняет BOOTSTRAP автоматически.
</identity>

<call_interface>
Каждый вызов должен содержать:
  agent_name:  имя вызывающего агента (обязательно)
  operation:   BOOTSTRAP | READ-CONTEXT | WRITE-ARTIFACT | UPSERT-KNOWLEDGE | UPDATE-STATUS
  task_id:     идентификатор задачи (обязателен для всех операций кроме BOOTSTRAP и UPSERT-KNOWLEDGE)

При отсутствии agent_name — отклонить с ошибкой: "agent_name обязателен".
</call_interface>

<bootstrap>

## Когда выполнять

BOOTSTRAP выполняется АВТОМАТИЧЕСКИ при любой операции, если .memory-bank/ не существует.
Явный вызов: operation: BOOTSTRAP без других параметров.

Порядок проверки перед любой операцией:
  1. Проверить существование .memory-bank/index.md
  2. Если не существует → выполнить BOOTSTRAP, затем продолжить исходную операцию
  3. Если существует → продолжить исходную операцию без BOOTSTRAP

## Шаги развёртывания

ШАГ 1 — Создать скелет директорий:
  .memory-bank/
  .memory-bank/mbb/
  .memory-bank/_common/
  .memory-bank/spec/
  .memory-bank/spec/process/
  .memory-bank/spec/skills/
  .memory-bank/spec/engineering/
  .memory-bank/spec/agents/
  .memory-bank/tasks/

ШАГ 2 — Создать обязательные файлы (шаблоны ниже):
  .memory-bank/index.md
  .memory-bank/mbb/principles.md
  .memory-bank/system-env.md
  .memory-bank/_common/index.md
  .memory-bank/_common/README.md

ШАГ 3 — Скопировать spec-файлы из .qwen/skills/ и .qwen/agents/ в .memory-bank/spec/:
  Если .qwen/ существует:
    .qwen/skills/memory-bank-owner/SKILL.md  → .memory-bank/spec/skills/memory-bank-owner.md
    .qwen/skills/commit-manager/SKILL.md     → .memory-bank/spec/skills/commit-manager.md
    .qwen/skills/system-env/SKILL.md         → .memory-bank/spec/skills/system-env.md
    .qwen/agents/*.md                        → .memory-bank/spec/agents/*.md
  Если .qwen/ не существует: пропустить ШАГ 3, зафиксировать в BOOTSTRAP-report.

ШАГ 4 — Вызвать skill system-env (operation: BOOTSTRAP, force: true)
  Записать результат в .memory-bank/system-env.md

ШАГ 5 — Вернуть BOOTSTRAP-report координатору (шаблон ниже).

## Шаблоны файлов для ШАГ 2

### .memory-bank/index.md
```
---
type: index
version: 1.0.0
---
# Memory Bank — Navigation Index

## Core
- [Principles](.memory-bank/mbb/principles.md)
- [System Environment](.memory-bank/system-env.md)

## Process
- [Pipeline](.memory-bank/spec/process/pipeline.md)
- [Status Protocol](.memory-bank/spec/process/status-protocol.md)

## Skills
- [memory-bank-owner](.memory-bank/spec/skills/memory-bank-owner.md)
- [commit-manager](.memory-bank/spec/skills/commit-manager.md)
- [system-env](.memory-bank/spec/skills/system-env.md)

## Engineering Standards
- [Quality Gate](.memory-bank/spec/engineering/quality-gate.md)
- [Guardrails](.memory-bank/spec/engineering/guardrails.md)
- [Testing Standards](.memory-bank/spec/engineering/testing-standards.md)

## Agents
- [planner](.memory-bank/spec/agents/planner.md)
- [spec-writer](.memory-bank/spec/agents/spec-writer.md)
- [architect](.memory-bank/spec/agents/architect.md)
- [tech-lead](.memory-bank/spec/agents/tech-lead.md)
- [unit-test-writer](.memory-bank/spec/agents/unit-test-writer.md)
- [developer](.memory-bank/spec/agents/developer.md)
- [code-reviewer](.memory-bank/spec/agents/code-reviewer.md)
- [integration-test-writer](.memory-bank/spec/agents/integration-test-writer.md)
- [qa-validator](.memory-bank/spec/agents/qa-validator.md)
- [docs-writer](.memory-bank/spec/agents/docs-writer.md)
- [release-manager](.memory-bank/spec/agents/release-manager.md)

## Technology Guides
- [Index](.memory-bank/_common/index.md)

## Project Tasks
<!-- Обновляется memory-bank-owner при создании новых задач -->
```

### .memory-bank/mbb/principles.md
Содержимое: скопировать из .memory-bank/spec/skills/memory-bank-owner.md секцию <bootstrap> → шаблон principles.md (см. файл в архиве).
Если файл недоступен: создать минимальный файл:
```
---
type: principles
version: 1.0.0
---
# Memory Bank Principles

Любое знание, не зафиксированное в memory-bank, не существует для агентов.
Запись — только через skill memory-bank-owner.
Чтение — напрямую любым агентом.
```

### .memory-bank/system-env.md
```
---
type: system-env
generated_by: skill/system-env
generated_at: UNSET
ttl_hours: 1
---
# System Environment
<!-- Заполняется skill system-env -->
```

### .memory-bank/_common/index.md
```
---
type: tech-guides-index
version: 1.0.0
---
# Technology Guides Index
<!-- Гайды добавляются через UPSERT-KNOWLEDGE -->
```

### .memory-bank/_common/README.md
```
---
type: meta
---
# Technology Guides — How to Add
Операция: UPSERT-KNOWLEDGE через skill memory-bank-owner.
path: .memory-bank/_common/<technology>/guide.md
Структура гайда: Problem / Solution / Example / Enforcement
После добавления — обновить _common/index.md через WRITE-ARTIFACT.
```

## BOOTSTRAP-report шаблон

```
BOOTSTRAP COMPLETE
  created_dirs:   <список созданных директорий>
  created_files:  <список созданных файлов>
  spec_copied:    true | false
  system_env:     populated | skipped
  warnings:       <список предупреждений или "none">
next_step: <следующая операция, прерванная bootstrap-ом, или "ready">
```

## Повторный BOOTSTRAP (repair)

Если .memory-bank/ существует, но повреждён (отсутствуют обязательные файлы):
  - Создать только недостающие файлы (не перезаписывать существующие).
  - Зафиксировать восстановленные файлы в BOOTSTRAP-report.
  - Признак повреждения: отсутствует index.md или mbb/principles.md.

</bootstrap>

<operations>

READ-CONTEXT
  Назначение: получить артефакты задачи для передачи субагенту.
  Параметры:
    task_id:   идентификатор задачи
    files:     список имён файлов (или "all" для всех артефактов задачи)
  Возвращает: содержимое запрошенных файлов.
  Если файл не существует: вернуть null для этого файла, не прерывать операцию.

WRITE-ARTIFACT
  Назначение: записать артефакт, возвращённый субагентом.
  Параметры:
    task_id:   идентификатор задачи
    filename:  имя файла (из списка допустимых артефактов)
    content:   содержимое файла
  Перед записью:
    1. Проверить существование .memory-bank/ → если нет, выполнить BOOTSTRAP.
    2. Проверить, что filename входит в допустимые артефакты (см. <artifacts>).
    3. Если создаётся новая задача — зарегистрировать в index.md секция "Project Tasks".
    4. Записать файл в .memory-bank/tasks/<task_id>/<filename>.
  После записи: подтвердить успех с путём файла.

UPDATE-STATUS
  Назначение: обновить status.md задачи.
  Параметры:
    task_id:   идентификатор задачи
    phase:     текущий этап конвейера
    agent:     последний выполнявший агент
    status:    in-progress | blocked | pending-review | completed
    blockers:  список блокеров (опционально)
    progress:  обновления строк таблицы прогресса (опционально)
  Записывает обновление в status.md без перезаписи незатронутых полей.

UPSERT-KNOWLEDGE
  Назначение: добавить или обновить tech-guide в _common/.
  Параметры:
    path:      .memory-bank/_common/<technology>/guide.md
    content:   Problem / Solution / Example / Enforcement
  После записи: обновить .memory-bank/_common/index.md ссылкой на гайд.

</operations>

<artifacts>
Допустимые имена файлов для WRITE-ARTIFACT:

  plan.md            — цель, скоуп, критерии успеха, декомпозиция
  status.md          — статус задачи, фаза, таблица прогресса
  spec.md            — user stories, acceptance criteria
  architecture.md    — контракты, ADR, структура
  dev-guide.md       — команды, conventions, стек
  review.md          — замечания code-reviewer
  qa-report.md       — отчёт qa-validator

Файлы вне этого списка — только при allow_custom: true.
</artifacts>

<templates>

plan.md:
```
---
task_id: <TASK-ID>
type: <NEWPROJECT|MAJORCHANGE|FEATURE|BUG|TECHDEBT|DOCS>
created_by: coordinator
---
# Plan: <название>

## Запрос пользователя
<исходный текст>

## Цель
<одно предложение>

## Скоуп
### Включено
- ...
### Исключено
- ...

## Критерии успеха
- [ ] ...

## Декомпозиция
- [ ] <этап>
```

status.md:
```
---
task_id: <TASK-ID>
status: in-progress
phase: planning
last_agent: coordinator
updated_at: <timestamp>
---
# Status: <TASK-ID>

## Прогресс по этапам

| Этап | Агент | Статус | Артефакт |
|------|-------|--------|----------|
| 1. planner | planner | pending | plan.md |
| 2. spec-writer | spec-writer | pending | spec.md |
| 3. architect | architect | pending | architecture.md |
| 4. tech-lead | tech-lead | — | dev-guide.md |
| 5. unit-test-writer | unit-test-writer | pending | тесты |
| 6. developer | developer | pending | реализация |
| 7. code-reviewer | code-reviewer | pending | review.md |
| 8. integration-test-writer | integration-test-writer | pending | тесты |
| 9. code-reviewer | code-reviewer | pending | review.md |
| 10. qa-validator | qa-validator | pending | qa-report.md |
| 11. docs-writer | docs-writer | pending | документация |
| 12. release-manager | release-manager | pending | — |

## Блокеры
- нет

## История
- <timestamp> coordinator: задача создана
```

</templates>
