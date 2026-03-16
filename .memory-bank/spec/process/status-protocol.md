---
type: process
version: 1.0.0
---
# Status Protocol

<rule>
Каждый субагент ОБЯЗАН вернуть явный статус из своего допустимого набора.
Координатор выполняет действие из таблицы ниже — без интерпретации.
Игнорирование не-успешного статуса ЗАПРЕЩЕНО.
</rule>

<table>
| Агент | Успех | Не-успех | Действие при не-успехе |
|-------|-------|----------|------------------------|
| planner | DONE | BLOCKED, QUESTION | Эскалировать пользователю |
| spec-writer | DONE | BLOCKED, QUESTION | Эскалировать пользователю |
| architect | DONE | BLOCKED | Вернуть к spec-writer или эскалировать |
| architect | DONE | QUESTION | Предложить альтернативы пользователю |
| tech-lead | DONE | BLOCKED | Запросить стек или конфиг у пользователя |
| tech-lead | DONE | QUESTION | Предложить альтернативы пользователю |
| unit-test-writer | DONE | BLOCKED | Вернуть к spec-writer или architect |
| developer | DONE | BLOCKED | Вернуть к architect или spec-writer |
| developer | DONE | PARTIAL | Зафиксировать частичный результат, запросить уточнение у пользователя |
| code-reviewer | APPROVE | REQUEST_CHANGES | Вернуть к developer с конкретным списком замечаний |
| code-reviewer | APPROVE | BLOCKED | Восстановить недостающий артефакт, повторить ревью |
| integration-test-writer | DONE | BLOCKED | Вернуть к architect или developer |
| qa-validator | PASS | FAIL | Вернуть к developer с перечнем непокрытых случаев |
| qa-validator | PASS | BLOCKED | Восстановить недостающий артефакт |
| docs-writer | DONE, NOT_NEEDED | BLOCKED | Восстановить недостающий артефакт |
| release-manager | DONE | BLOCKED | Вернуть на этап, породивший блокер |
| release-manager | DONE | CONFLICT | Вернуть к developer |
</table>

<escalation_triggers>
Немедленно эскалировать пользователю (остановить конвейер) при:
- BLOCKED или QUESTION от planner или spec-writer
- Один и тот же BLOCKED повторяется дважды подряд у одного агента
- BLOCKED или CONFLICT от release-manager
- Любой агент не возвращает статус после выполнения
</escalation_triggers>

<status_definitions>
DONE          — задача выполнена полностью, артефакт готов
APPROVE       — код соответствует всем требованиям, замечаний нет
PASS          — качество подтверждено, покрытие достаточно
NOT_NEEDED    — документация не требуется (с обоснованием)
BLOCKED       — невозможно продолжить без внешней информации или артефакта
QUESTION      — требуется решение пользователя по неоднозначному требованию
PARTIAL       — выполнено частично, перечень невыполненного прилагается
REQUEST_CHANGES — есть конкретные замечания, требующие исправления
FAIL          — проверка не пройдена, перечень проблем прилагается
CONFLICT      — конфликт в коде или ветке, требует ручного разрешения
</status_definitions>
