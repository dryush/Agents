---
type: process
version: 1.1.0
---
# Pipeline

<entry_points>
Точка входа определяется типом задачи:

  NEWPROJECT, MAJORCHANGE → ЭТАП 1 (planner)
  FEATURE                 → ЭТАП 2 (spec-writer)
  BUG                     → ЭТАП 3 (architect, триаж)
  TECHDEBT                → ЭТАП 3 (architect)
  DOCS                    → ЭТАП 11 (docs-writer, без TDD-цикла)
  OTHER                   → ЭТАП 1 (planner)
</entry_points>

<stages>
ЭТАП 1 — planner
  ВХОД:    запрос пользователя
  ВЫХОД:   plan.md (статус DRAFT), status.md
  GATE:    цель и критерии успеха определены, скоуп зафиксирован,
           иерархия epic → feature задокументирована
  ВАЖНО:   секция «Под-задачи разработки» в plan.md остаётся DRAFT до ЭТАПА 3.
           Planner НЕ детализирует под-задачи реализации — это зона architect.

ЭТАП 2 — spec-writer
  ВХОД:    plan.md существует
  ВЫХОД:   spec.md создан
  GATE:    все acceptance criteria в проверяемом формате, открытые вопросы явно зафиксированы

ЭТАП 3 — architect
  ВХОД:    spec.md существует
  ВЫХОД:   architecture.md, контракты модулей
  GATE:    все затронутые модули описаны, стратегия тестирования зафиксирована
  ДОПОЛНИТЕЛЬНОЕ ДЕЙСТВИЕ:
    После создания architecture.md architect формирует и передаёт координатору
    список под-задач разработки:
      - имя под-задачи (класс / группа методов)
      - зависимости (имена других под-задач из этого же списка)
      - принадлежность к feature и epic из plan.md
    Координатор вносит список в секцию «Под-задачи разработки» в plan.md.
    Координатор устанавливает plan.md статус: FINAL.
    TDD-цикл не может начаться, пока plan.md не имеет статус FINAL.

ЭТАП 4 — tech-lead  [ОПЦИОНАЛЬНЫЙ]
  УСЛОВИЕ ВЫЗОВА: отсутствует dev-guide.md ИЛИ меняется стек ИЛИ меняется инфраструктура
  ВХОД:    architecture.md существует
  ВЫХОД:   dev-guide.md создан или обновлён
  GATE:    команды lint/test/build зафиксированы, изоляция окружения настроена

─── TDD-ЦИКЛ ──────────────────────────────────────────────

ЭТАП 5 — unit-test-writer
  ВХОД:    architecture.md, dev-guide.md существуют, plan.md имеет статус FINAL
  ВЫХОД:   dependency-map.md, tdd-task-list.md (декомпозиция — однократно, в первом запуске),
           unit-тесты текущей под-задачи, статус RED подтверждён
  GATE:    покрыты happy path, alt path, edge cases, error cases по каждому контракту

ЭТАП 6 — developer
  ВХОД:    тесты RED существуют, все зависимости текущей под-задачи DONE
  ВЫХОД:   реализация модуля, тесты GREEN, quality gate зелёный
  GATE:    100% тестов модуля проходят, ноль пропущенных
  ПРИ IMPL_BLOCKED: см. протокол IMPL_BLOCKED в AGENTS.md <pipeline>
  ПРИ ARCH_REVISION:
    developer возвращает ARCH_REVISION с описанием архитектурной проблемы.
    Координатор немедленно останавливает TDD-цикл.
    Конвейер перезапускается с ЭТАПА 3 (architect).
    Текущие тесты и tdd-task-list.md сохраняются.
    После возврата architect пересматривает architecture.md и обновляет список
    под-задач (координатор обновляет plan.md → FINAL повторно).
    unit-test-writer пересматривает dependency-map.md и tdd-task-list.md.
    TDD-цикл возобновляется с первой затронутой под-задачи.

ЭТАП 7 — code-reviewer
  ВХОД:    реализация и тесты GREEN
  ВЫХОД:   APPROVE или REQUEST_CHANGES
  GATE:    вердикт APPROVE
  ПРИ REQUEST_CHANGES: → вернуться к ЭТАП 6
  УСЛОВИЕ ВЫХОДА ИЗ ЦИКЛА: code-reviewer вернул APPROVE

─── КОНЕЦ TDD-ЦИКЛА ────────────────────────────────────────

ЭТАП 8 — integration-test-writer
  ВХОД:    реализация APPROVE от code-reviewer
  ВЫХОД:   интеграционные и e2e тесты
  GATE:    сценарии взаимодействия модулей покрыты, внешние зависимости замоканы

ЭТАП 9 — code-reviewer (повторный)
  ВХОД:    интеграционные тесты существуют
  ВЫХОД:   APPROVE или REQUEST_CHANGES
  GATE:    вердикт APPROVE
  ПРИ REQUEST_CHANGES: → вернуться к ЭТАП 8

ЭТАП 10 — qa-validator
  ВХОД:    APPROVE от code-reviewer (этап 9)
  ВЫХОД:   PASS или FAIL
  GATE:    вердикт PASS
  ПРИ FAIL: → вернуться к ЭТАП 6

ЭТАП 11 — docs-writer
  ВХОД:    qa-validator PASS
  ВЫХОД:   DONE или NOT_NEEDED
  GATE:    DONE или NOT_NEEDED с обоснованием

ЭТАП 12 — release-manager
  ВХОД:    все предыдущие этапы завершены успешно
  ВЫХОД:   DONE (merge выполнен) или BLOCKED/CONFLICT
  GATE:    полный release-checklist зелёный (см. AGENTS.md <release_gate>)
</stages>

<artifacts>
Обязательные артефакты по задаче (путь: .memory-bank/tasks/<TASK-ID>/):

  plan.md           — цель, скоуп, критерии успеха, иерархия epic→feature,
                      под-задачи разработки (DRAFT → FINAL после ЭТАПА 3)
  status.md         — текущий статус, фаза, таблица прогресса по этапам
  spec.md           — user stories, acceptance criteria, пути выполнения
  architecture.md   — контракты, ADR, структура файлов, стратегия тестирования
  dev-guide.md      — команды, conventions, стек (может быть общим для проекта)
  dependency-map.md — граф зависимостей под-задач, test-scope map (создаёт unit-test-writer)
  tdd-task-list.md  — упорядоченный список под-задач со статусами (создаёт unit-test-writer)
  review.md         — замечания code-reviewer (создаётся при REQUEST_CHANGES)
  qa-report.md      — отчёт qa-validator
</artifacts>

<plan_md_format>
Обязательные секции plan.md:

  ## Цель
  ## Критерии успеха
  ## Скоуп (входит / не входит)
  ## Эпики и фичи
    - epic-name
      - feature-name
      - feature-name
  ## Под-задачи разработки  [статус: DRAFT | FINAL]
    Заполняется координатором по данным architect после ЭТАПА 3.
    Формат: таблица (под-задача | зависимости | feature | epic)
  ## Открытые вопросы
</plan_md_format>

<task_lifecycle>
Статусы задачи в status.md:

  planning → in-progress → pending-review → completed
                          ↓
                       blocked (при BLOCKED от любого агента)
                          ↓
                   arch-revision (при ARCH_REVISION от developer)
                       ↓
                    in-progress (после DONE от architect повторно)

Координатор обновляет status.md после каждого этапа.
</task_lifecycle>

<question_ownership>
Таблица: кто задаёт вопросы о чём

  Вопрос о...                              | Кто отвечает
  -----------------------------------------|-------------------
  Цель, ценность, скоуп задачи             | planner → пользователь
  Ожидаемое поведение, бизнес-правила      | spec-writer → пользователь
  Параметры UX (размер, скорость, лимиты)  | spec-writer → пользователь
  Язык, фреймворк, библиотеки              | architect (решает сам, ADR)
  Структура модулей, паттерны              | architect (решает сам, ADR)
  Стратегия тестирования                   | architect (решает сам)
  Инфраструктура, CI/CD, окружение         | tech-lead (решает сам)
  Декомпозиция под-задач (финальная)       | architect (решает сам, вносит в plan.md)
  Архитектурная проблема в ходе реализации | developer → ARCH_REVISION → architect

Правило: агент не эскалирует вопросы, которые входят в его зону.
Он принимает решение самостоятельно и фиксирует его в ADR или артефакте.
Пользователь получает вопрос ТОЛЬКО если агент не может принять решение
без бизнес-контекста, которого у него нет.
</question_ownership>

<arch_revision_protocol>
Инициируется developer при обнаружении в ходе реализации проблемы,
которая делает текущую архитектуру невозможной или некорректной.

Критерии применения ARCH_REVISION (ВСЕ должны выполняться):
  1. Проблема не решается рефакторингом в рамках существующих контрактов.
  2. Решение требует изменения интерфейсов, зависимостей или структуры модулей.
  3. Продолжение реализации без решения создаст технический долг или баг.

Не является ARCH_REVISION:
  - Стилистические разногласия
  - Вопросы именования без влияния на контракты
  - Неясность в реализации, решаемая запросом к spec.md или architecture.md
  - Противоречие тестов и контрактов (для этого есть IMPL_BLOCKED)

Порядок действий при ARCH_REVISION:
  1. developer возвращает координатору:
       status: ARCH_REVISION
       subtask: <имя текущей под-задачи>
       problem: <описание проблемы: что именно не работает и почему>
       affected_contracts: <список затронутых контрактов/интерфейсов>
  2. Координатор:
       a. Останавливает TDD-цикл.
       b. Фиксирует статус задачи arch-revision в status.md.
       c. Сохраняет текущее состояние tdd-task-list.md (статусы не сбрасываются).
       d. Делегирует architect с передачей: task_id, subtask, problem, affected_contracts.
  3. architect:
       a. Читает architecture.md и описание проблемы.
       b. Пересматривает затронутые контракты.
       c. Обновляет architecture.md.
       d. Формирует обновлённый список под-задач (дельта: что изменилось).
       e. Возвращает DONE.
  4. Координатор:
       a. Обновляет секцию «Под-задачи разработки» в plan.md → FINAL.
       b. Делегирует unit-test-writer для обновления dependency-map.md и tdd-task-list.md.
       c. Возобновляет TDD-цикл с первой затронутой под-задачи.
  5. Все этапы после architect (tech-lead при необходимости, unit-test-writer,
     developer, code-reviewer) повторяются для затронутых под-задач.

Ограничение: ARCH_REVISION не может инициировать никто, кроме developer.
code-reviewer, qa-validator и другие агенты при обнаружении архитектурной
проблемы фиксируют её в замечаниях и возвращают REQUEST_CHANGES/FAIL —
developer решает, требует ли это ARCH_REVISION.
</arch_revision_protocol>
