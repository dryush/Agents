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
  .memory-bank/adr/
  .memory-bank/_common/
  .memory-bank/spec/
  .memory-bank/spec/process/
  .memory-bank/spec/engineering/
  .memory-bank/spec/agents/
  .memory-bank/tasks/

ШАГ 2 — Создать обязательные файлы из шаблонов:
  Источник шаблонов: .qwen/skills/memory-bank-owner/templates/

  templates/index.md          → .memory-bank/index.md
  templates/principles.md     → .memory-bank/mbb/principles.md
  templates/system-env.md     → .memory-bank/system-env.md
  templates/_common-index.md  → .memory-bank/_common/index.md
  templates/_common-README.md → .memory-bank/_common/README.md
  templates/adr-index.md      → .memory-bank/adr/index.md

ШАГ 3 — Скопировать инструкции агентов:
  Если .qwen/agents/ существует:
    .qwen/agents/*.md → .memory-bank/spec/agents/*.md
  Если .qwen/agents/ не существует: пропустить, зафиксировать в BOOTSTRAP-report.

ШАГ 4 — Активировать skill system-env (operation: BOOTSTRAP, force: true)
  Записать результат в .memory-bank/system-env.md

ШАГ 5 — Вернуть BOOTSTRAP-report (шаблон ниже).

## BOOTSTRAP-report шаблон

```
BOOTSTRAP COMPLETE
  created_dirs:   <список созданных директорий>
  created_files:  <список созданных файлов>
  agents_copied:  true | false
  system_env:     populated | skipped
  warnings:       <список предупреждений или "none">
next_step: <следующая операция или "ready">
```

## Повторный BOOTSTRAP (repair)

Если .memory-bank/ существует, но повреждён (отсутствуют обязательные файлы):
  - Создать только недостающие файлы из шаблонов (не перезаписывать существующие).
  - Зафиксировать восстановленные файлы в BOOTSTRAP-report.
  - Признаки повреждения: отсутствует index.md или mbb/principles.md.

</bootstrap>

<operations>

READ-CONTEXT
  Назначение: получить артефакты задачи для субагента.
  Параметры:
    task_id:   идентификатор задачи
    files:     список имён файлов (или "all" для всех артефактов задачи)
  Возвращает: содержимое запрошенных файлов.
  Если файл не существует: вернуть null для этого файла, не прерывать операцию.

WRITE-ARTIFACT
  Назначение: записать артефакт субагента.
  Параметры:
    task_id:   идентификатор задачи
    filename:  имя файла (из списка допустимых артефактов)
    content:   содержимое файла
  Перед записью:
    1. Проверить существование .memory-bank/ → если нет, выполнить BOOTSTRAP.
    2. Проверить, что filename входит в допустимые артефакты (см. <artifacts>).
    3. Если создаётся новая задача — зарегистрировать в index.md (секция "Project Tasks").
       Использовать шаблоны/status.md для первичного status.md задачи.
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
  architecture.md    — контракты, tech decision matrix, конфигурационный contract, модель изоляции
  dev-guide.md       — команды setup/run/test/build, conventions, примеры конфигурации
  review.md          — замечания code-reviewer
  qa-report.md       — отчёт qa-validator
  adr-N.md           — task-level ADR (N = порядковый номер внутри задачи)

Глобальные ADR пишутся напрямую в .memory-bank/adr/adr-NNN-<slug>.md
и регистрируются в .memory-bank/adr/index.md.

Файлы вне этого списка — только при allow_custom: true.
</artifacts>
