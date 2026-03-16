---
name: branch-manager
description: Управляет ветками репозитория. MUST BE USED при создании новой задачи (CREATE-BRANCH) и при её завершении (CLEANUP). Use PROACTIVELY — ни один агент не создаёт ветки напрямую.
---
# Skill: branch-manager

<identity>
Единственная точка управления ветками git.
Обеспечивает соответствие структуры веток структуре задач.
Принимает запросы только с явным agent_name и task_id.
</identity>

<call_interface>
  agent_name:  имя вызывающего агента (обязательно)
  operation:   CREATE-BRANCH | GET-BRANCH-NAME | CLEANUP
  task_id:     идентификатор задачи (обязательно)
  parent_id:   идентификатор родительской задачи или эпика (опционально)
</call_interface>

<naming_policy>
## Формат имени ветки

  <prefix>/<task-id>[-<slug>]

Префиксы по типу задачи:
  feature/   — FEATURE, NEWPROJECT, MAJORCHANGE
  fix/        — BUG
  refactor/  — TECHDEBT
  docs/      — DOCS
  chore/     — инфраструктура, конфиг

Slug — kebab-case из названия задачи, максимум 4 слова.
Примеры:
  feature/FT-0042-user-auth
  fix/BG-0017-null-pointer
  refactor/TD-0003-extract-service
  docs/DC-0001-api-reference

## Иерархия веток (эпики)

Если task_id принадлежит эпику (EP-XXX):
  Эпик:       feature/EP-001-auth-system
  Подзадача:  feature/EP-001-FT-042-login-form

Ветка подзадачи ответвляется от ветки эпика, не от main.
Если ветка эпика не существует — создать её первой.

## Защищённые ветки (неприкосновенны)
  main, master, develop, staging, production
  Прямые коммиты в эти ветки — ЗАПРЕЩЕНЫ.
  Изменения только через PR/merge от release-manager.
</naming_policy>

<operations>

CREATE-BRANCH
  Назначение: создать ветку для новой задачи.
  Параметры:
    task_id:    идентификатор задачи
    task_type:  тип задачи (FEATURE|BUG|TECHDEBT|DOCS|...)
    task_slug:  краткое название (1-4 слова, kebab-case)
    parent_id:  идентификатор эпика (если подзадача)
    base:       базовая ветка (по умолчанию: develop или main)
  Действия:
    1. Сформировать имя ветки по naming_policy.
    2. Проверить существование .git/:
       - если НЕТ → выполнить git init, зафиксировать в отчёте (git_initialized: true)
       - если ДА → продолжить
    3. Проверить, что base не является защищённой (если тип не release).
    4. Если base-ветка существует: git checkout <base> && git pull
       Если репозиторий только что инициализирован (шаг 2): пропустить checkout base.
    5. Выполнить: git checkout -b <branch-name>
    6. Зафиксировать branch_name в .memory-bank/tasks/<task_id>/status.md
       через memory-bank-owner (UPDATE-STATUS, поле branch).
  Возвращает: branch_name, base_branch, git_initialized, выполненные команды.

GET-BRANCH-NAME
  Назначение: получить имя ветки для существующей задачи.
  Параметры:
    task_id:  идентификатор задачи
  Действия: прочитать поле branch из status.md задачи.
  Возвращает: branch_name или null если ветка не создана.

CLEANUP
  Назначение: удалить ветку задачи после успешного merge.
  Параметры:
    task_id:    идентификатор задачи
    remote:     true | false (удалить remote-ветку, по умолчанию false)
  Условие: выполнять только после подтверждения merge от release-manager.
  Действия:
    1. Получить branch_name через GET-BRANCH-NAME.
    2. Через run_shell_command:
         a. git checkout develop 2>&1 || git checkout main 2>&1
         b. git branch -d <branch-name> 2>&1
         c. Если remote: true → git push origin --delete <branch-name> 2>&1
  Возвращает: статус удаления.

</operations>

<rules>
- Создание ветки без task_id — ЗАПРЕЩЕНО.
- Только branch-manager выполняет git-команды для веток. Все остальные агенты —
  координатор, субагенты — не вызывают git checkout / git branch / git init напрямую.
- Коммит в ветку не своей задачи — ЗАПРЕЩЕНО (проверяется commit-manager).
- git push в защищённую ветку — ЗАПРЕЩЕНО.
- Ветка задачи создаётся один раз — при старте задачи координатором.
- Если репозиторий не инициализирован — branch-manager выполняет git init самостоятельно,
  не делегируя это координатору или субагентам.
</rules>
