---
name: commit-manager
description: Единственная точка создания git-коммитов. MUST BE USED перед каждым коммитом. Проверяет quality gate, формирует сообщение по шаблону с agent_name и task_id.
---

# Skill: commit-manager

<identity>
Единственная точка создания git-коммитов.
Принимает запросы только с явным agent_name и task_id.
Без обоих параметров — коммит запрещён.
</identity>

<call_interface>
Перед выполнением определи следующие параметры:
  agent_name:  имя вызывающего агента (обязательно)
  task_id:     идентификатор задачи (обязательно)
  phase:       этап конвейера (обязательно)
  files:       список файлов для включения в коммит
  message:     тело сообщения (опционально, дополняет шаблон)
</call_interface>

<commit_format>
Шаблон сообщения коммита:

  <type>(<scope>): <описание>

  [TASK-ID: <task_id>]
  [agent: <agent_name>]
  [phase: <phase>]

Типы:
  feat     — новая функциональность
  fix      — исправление бага
  test     — добавление или изменение тестов
  refactor — рефакторинг без изменения поведения
  docs     — документация
  chore    — инфраструктура, конфиг, зависимости

TDD-фазы как отдельные коммиты:
  test(scope): add unit tests [RED]   — unit-test-writer
  feat(scope): implement <feature>    — developer (GREEN)
  refactor(scope): clean up <feature> — developer (REFACTOR, если применимо)
</commit_format>

<rules>
- Коммит без agent_name — ЗАПРЕЩЁН.
- Коммит без task_id — ЗАПРЕЩЁН.
- Коммит с флагом --no-verify — ЗАПРЕЩЁН.
- Коммит с незелёным quality gate — ЗАПРЕЩЁН (проверяется перед коммитом).
- git push выполняется только при явном разрешении пользователя.
- Merge в main/master — только release-manager с полным чеклистом.
</rules>

<branch_check>
Перед коммитом проверить:
  1. Текущая ветка (git branch --show-current) соответствует task_id из вызова.
     Ожидаемое имя — через branch-manager (GET-BRANCH-NAME, task_id).
  2. Если ветка не совпадает — ОТКАЗАТЬ в коммите с ошибкой:
     "COMMIT REJECTED: текущая ветка <X> не соответствует задаче <task_id> (ожидалась <Y>)"
  3. Коммит в защищённую ветку (main, master, develop, staging, production) — ЗАПРЕЩЁН.
     Исключение: явная команда release-manager с параметром allow_protected: true.
</branch_check>
