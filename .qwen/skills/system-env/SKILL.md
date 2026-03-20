---
name: system-env
description: Собирает и кеширует данные об окружении (OS, рантаймы, пакетные менеджеры). Use PROACTIVELY при первом запуске в проекте или после смены машины. TTL кеша 1 час.
---

# Skill: system-env

<identity>
Собирает данные об окружении и кеширует в .memory-bank/system-env.md.
TTL кеша: 1 час. Если кеш свежее 1 часа — не обновлять, вернуть путь к кешу.
</identity>

<call_interface>
Перед выполнением определи следующие параметры:
  agent_name:  имя вызывающего агента (обязательно)
  force:       true — принудительно обновить кеш (опционально)
  current_time: текущее время
</call_interface>

<collection>
Собирать командами, адаптированными под текущую платформу:

  platform:
    - операционная система и версия
    - архитектура процессора
    - текущая оболочка (shell) и её версия

  runtimes (проверять наличие каждого):
    - node / bun / deno (версия)
    - python / python3 (версия)
    - java / kotlin (версия)
    - go / rust (версия)
    - ruby (версия)

  package_managers (проверять наличие каждого):
    - npm / yarn / pnpm / bun
    - pip / poetry / uv
    - cargo / gradle / maven

  vcs:
    - git (версия)
    - текущая ветка репозитория (если применимо)

  notes:
    - переменные окружения, влияющие на сборку (PATH, JAVA_HOME и т.п.)
    - ограничения окружения (нет интернета, sandbox, и т.п.)
</collection>

<output_format>
Записать результат в .memory-bank/system-env.md в формате:

```
---
type: system-env
generated_by: skill/system-env
generated_at: <ISO-8601 timestamp>
ttl_hours: 1
---
# System Environment

## Platform
- os: <значение>
- arch: <значение>
- shell: <значение>

## Runtimes
- <runtime>: <версия или "not found">

## Package Managers
- <pm>: <версия или "not found">

## VCS
- git: <версия>
- branch: <текущая ветка или "not a git repo">

## Notes
- <особенности окружения>
```

После записи: вернуть путь .memory-bank/system-env.md и краткое резюме платформы.
</output_format>

<adaptation_rule>
Все агенты, использующие shell-команды, ОБЯЗАНЫ прочитать system-env.md
и адаптировать команды под зафиксированную платформу и оболочку.
</adaptation_rule>
