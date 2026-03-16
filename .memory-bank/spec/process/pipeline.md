---
type: process
version: 1.0.0
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
  ВЫХОД:   plan.md, status.md созданы
  GATE:    цель и критерии успеха определены, скоуп зафиксирован

ЭТАП 2 — spec-writer
  ВХОД:    plan.md существует
  ВЫХОД:   spec.md создан
  GATE:    все acceptance criteria в проверяемом формате, открытые вопросы явно зафиксированы

ЭТАП 3 — architect
  ВХОД:    spec.md существует
  ВЫХОД:   architecture.md, контракты модулей
  GATE:    все затронутые модули описаны, стратегия тестирования зафиксирована

ЭТАП 4 — tech-lead  [ОПЦИОНАЛЬНЫЙ]
  УСЛОВИЕ ВЫЗОВА: отсутствует dev-guide.md ИЛИ меняется стек ИЛИ меняется инфраструктура
  ВХОД:    architecture.md существует
  ВЫХОД:   dev-guide.md создан или обновлён
  GATE:    команды lint/test/build зафиксированы, изоляция окружения настроена

─── TDD-ЦИКЛ ──────────────────────────────────────────────

ЭТАП 5 — unit-test-writer
  ВХОД:    architecture.md и dev-guide.md существуют
  ВЫХОД:   unit-тесты, статус RED подтверждён
  GATE:    покрыты happy path, alt path, edge cases, error cases по каждому контракту

ЭТАП 6 — developer
  ВХОД:    тесты RED существуют
  ВЫХОД:   реализация, все тесты GREEN, quality gate зелёный
  GATE:    100% тестов проходят, ноль пропущенных, quality gate зелёный

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

  plan.md          — цель, скоуп, критерии успеха, декомпозиция
  status.md        — текущий статус, фаза, таблица прогресса по этапам
  spec.md          — user stories, acceptance criteria, пути выполнения
  architecture.md  — контракты, ADR, структура файлов, стратегия тестирования
  dev-guide.md     — команды, conventions, стек (может быть общим для проекта)
  review.md        — замечания code-reviewer (создаётся при REQUEST_CHANGES)
  qa-report.md     — отчёт qa-validator
</artifacts>

<task_lifecycle>
Статусы задачи в status.md:

  planning       → in-progress → pending-review → completed
                                ↓
                             blocked (при BLOCKED от любого агента)

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

Правило: агент не эскалирует вопросы, которые входят в его зону.
Он принимает решение самостоятельно и фиксирует его в ADR или артефакте.
Пользователь получает вопрос ТОЛЬКО если агент не может принять решение
без бизнес-контекста, которого у него нет.
</question_ownership>
