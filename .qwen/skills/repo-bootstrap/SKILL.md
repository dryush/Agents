---
name: repo-bootstrap
description: >
  Инициализация локального git-репозитория и первый коммит.
  MUST BE USED когда тип задачи NEWPROJECT и локальный git-репозиторий отсутствует.
  Создаёт git init, базовые ignore-файлы и выполняет первый коммит.
  Не определяет прикладной стек.
---

# Skill: repo-bootstrap

<identity>
Единственная точка инициализации нового git-репозитория.
</identity>

<call_interface>
Перед выполнением определи следующие параметры:
  agent_name:  имя вызывающего агента (обязательно)
  project_dir: путь к директории проекта (по умолчанию — текущая директория)
  description: краткое описание проекта для первого коммита (опционально)
</call_interface>

## Instructions

### Шаг 1. Проверить, что репозиторий отсутствует

```shell
git -C <project_dir> rev-parse --is-inside-work-tree 2>/dev/null
```

- Если вернул `true` — репозиторий уже существует. Вернуть статус `ALREADY_EXISTS`.
- Если команда завершилась с ошибкой — продолжать.

### Шаг 2. Инициализировать репозиторий

```shell
git init <project_dir>
```

Адаптировать команду под платформу из `system-env.md`.

### Шаг 3. Создать .gitignore

Создать `.gitignore` в корне проекта со стандартными правилами, не привязанными к стеку:

```
# OS
.DS_Store
Thumbs.db

# Editors
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log
logs/

# Environment
.env
.env.local
.env.*.local

# Runtime / build artifacts (stack-agnostic placeholders)
dist/
build/
out/
__pycache__/
*.pyc
node_modules/
.venv/
```

> Стек-специфичные записи добавляет tech-lead после выбора архитектором технологий.

### Шаг 4. Выполнить первый коммит

```shell
git -C <project_dir> add .
git -C <project_dir> commit -m "chore: init repository

[agent: <agent_name>]
[phase: repo-bootstrap]"
```

Флаг `--no-verify` ЗАПРЕЩЁН.

### Шаг 5. Вернуть статус

- `DONE` — репозиторий инициализирован, `.gitignore` создан, первый коммит сделан.
- `ALREADY_EXISTS` — репозиторий уже существует, инициализация пропущена.
- `BLOCKED` — невозможно выполнить `git init`, с описанием причины.

<rules>
- git init выполняется только если репозиторий отсутствует.
- Флаг --no-verify ЗАПРЕЩЁН.
- agent_name обязателен; без него вызов отклоняется.
</rules>
