---
type: engineering
version: 1.0.0
---
# TDD Modular Process

<purpose>
Определяет процесс декомпозиции задачи на атомарные под-задачи и порядок их
выполнения в рамках TDD-цикла с учётом внутри-проектных зависимостей.
</purpose>

<definitions>
  под-задача   — один класс или одна логически связанная группа методов
  зависимость  — под-задача B зависит от A, если реализация B требует
                 вызова или импорта сущностей из A
  DONE         — под-задача прошла ШАГ C (APPROVE от code-reviewer),
                 тесты модуля GREEN
  test-scope   — минимальный набор тест-файлов, связанных с конкретной под-задачей,
                 feature или epic
</definitions>

<phase_decomposition>
Выполняется unit-test-writer ОДИН РАЗ до начала итерационного TDD-цикла.
Входные данные: plan.md со статусом FINAL (секция «Под-задачи разработки» заполнена architect-ом).

ШАГ D1 — Анализ контрактов
  Прочитать architecture.md: все интерфейсы, классы, модули.
  Прочитать spec.md: acceptance criteria по каждой feature и epic.
  Прочитать plan.md: список под-задач от architect.

ШАГ D2 — Построение графа зависимостей
  Для каждой под-задачи определить список зависимостей.
  Построить направленный граф: ребро A→B означает «B зависит от A».
  Проверить отсутствие циклических зависимостей.
  Если цикл найден → вернуть BLOCKED с описанием цикла координатору.

ШАГ D3 — Топологическая сортировка
  Упорядочить под-задачи методом топологической сортировки (Kahn's algorithm или DFS).
  Под-задачи без зависимостей — первыми.
  Под-задачи, зависящие от незавершённых — блокированы до DONE зависимостей.

ШАГ D4 — Запись артефактов
  Записать dependency-map.md (граф зависимостей, формат: таблица или mermaid graph).
  Записать tdd-task-list.md (упорядоченный список под-задач, см. ниже).
  Оба артефакта — через memory-bank-owner (WRITE-ARTIFACT).
</phase_decomposition>

<artifact_dependency_map>
Файл: .memory-bank/tasks/<TASK-ID>/dependency-map.md

Структура:
  ## Граф зависимостей
  <mermaid или таблица: под-задача → зависит от>

  ## Test Scope Map
  Таблица: под-задача | test-scope (пути к тест-файлам) | feature | epic
  Пример:
  | Под-задача      | Test scope                           | Feature      | Epic   |
  |-----------------|--------------------------------------|--------------|--------|
  | UserRepository  | tests/unit/UserRepository.test.ts    | user-profile | users  |
  | UserService     | tests/unit/UserService.test.ts       | user-profile | users  |
  | AuthService     | tests/unit/AuthService.test.ts       | auth         | users  |

  ## Feature Test Scope
  Таблица: feature | test-файлы всех под-задач этой feature

  ## Epic Test Scope
  Таблица: epic | test-файлы всех feature этого epic
</artifact_dependency_map>

<artifact_tdd_task_list>
Файл: .memory-bank/tasks/<TASK-ID>/tdd-task-list.md

Структура:
  ## TDD Task List

  | # | Под-задача     | Зависит от     | Статус    | Test scope file                        |
  |---|----------------|----------------|-----------|----------------------------------------|
  | 1 | EmailValidator | —              | pending   | tests/unit/EmailValidator.test.ts      |
  | 2 | UserRepository | EmailValidator | pending   | tests/unit/UserRepository.test.ts      |
  | 3 | UserService    | UserRepository | pending   | tests/unit/UserService.test.ts         |

Статусы под-задачи:
  pending     — не начата
  in-progress — unit-test-writer написал тесты (RED подтверждён)
  done        — APPROVE от code-reviewer, тесты GREEN

Обновление статуса — через memory-bank-owner (UPDATE-STATUS) после каждого ШАГ C.
</artifact_tdd_task_list>

<execution_rules>
ПРАВИЛО 1 — Блокировка
  developer НЕ ПРИСТУПАЕТ к реализации под-задачи, если хотя бы одна зависимость
  имеет статус, отличный от DONE.
  Координатор проверяет tdd-task-list.md перед каждой делегацией developer.
  Если следующая под-задача заблокирована — выполняется предыдущая по порядку.

ПРАВИЛО 2 — Запуск тестов при реализации модуля
  developer запускает тест-команду ТОЛЬКО для test-scope текущей под-задачи.
  Запрещено запускать полный test suite в рамках реализации одного модуля.
  Команда: определяется инструментом test-scope-runner (реализует tech-lead).

ПРАВИЛО 3 — Smoke-прогон после завершения под-задачи
  После получения APPROVE от code-reviewer developer запускает:
  - тесты всех под-задач той же feature (feature test scope)
  Если хотя бы один тест упал → статус BLOCKED, зафиксировать регрессию.

ПРАВИЛО 4 — Epic smoke-прогон
  После завершения ВСЕХ под-задач одного epic developer запускает:
  - тесты всех feature затронутого epic (epic test scope)
  Если тест упал → BLOCKED с описанием регрессии.

ПРАВИЛО 5 — Полный прогон перед merge
  release-manager запускает полный test suite (quality gate ШАГ 3)
  ТОЛЬКО перед слиянием в main.
  Ни один другой агент не запускает полный suite.
</execution_rules>

<tooling_contract>
Инструменты ниже определяют ИНТЕРФЕЙС. Реализацию выполняет tech-lead
в dev-guide.md с учётом технологий конкретного проекта.

  test-scope-runner <scope-type> <target>
    scope-type: module | feature | epic | full
    target: имя модуля / feature / epic (соответствует dependency-map.md)
    Запускает только тест-файлы из соответствующего scope.
    exit 0 → все тесты прошли; exit 1 → есть падения.

  dependency-validator
    Читает dependency-map.md текущей задачи.
    Проверяет: нет циклов, все зависимости присутствуют в tdd-task-list.md.
    exit 0 → граф валиден; exit 1 → найдены ошибки с описанием.

  task-status-checker <task-id> <subtask-name>
    Читает tdd-task-list.md.
    Возвращает статус под-задачи: pending | in-progress | done.
    Используется координатором перед делегацией developer.

Документирование инструментов в dev-guide.md:
  - команда запуска с примерами
  - формат вывода (stdout)
  - exit codes
  - требования к окружению (локальное venv / node_modules и т.п.)
</tooling_contract>

<agent_responsibilities>
  unit-test-writer:
    - Выполняет phase_decomposition один раз (артефакты: dependency-map.md, tdd-task-list.md)
    - Для каждой под-задачи пишет тесты в RED-статусе
    - Обновляет статус под-задачи в tdd-task-list.md → in-progress

  developer:
    - Перед реализацией проверяет: все зависимости под-задачи имеют статус DONE
    - Запускает test-scope-runner module <subtask-name> после реализации
    - Запускает test-scope-runner feature <feature-name> после APPROVE
    - Запускает test-scope-runner epic <epic-name> после завершения всех под-задач epic
    - Обновляет статус под-задачи в tdd-task-list.md → done после APPROVE

  coordinator:
    - Перед каждой делегацией developer проверяет tdd-task-list.md:
      следующая под-задача имеет статус pending И все зависимости DONE
    - При обнаружении заблокированной под-задачи — выбирает следующую незаблокированную

  release-manager:
    - Запускает test-scope-runner full перед merge
    - Проверяет, что все под-задачи в tdd-task-list.md имеют статус DONE
</agent_responsibilities>
