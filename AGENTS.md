# AGENTS.md — Hermes Optimization Guide

Валидация **обязательна** перед каждым PR. Это документационный репозиторий — сломанные ссылки и невалидные скиллы блокируют CI.

## Проверка перед коммитом

```bash
# 1. Валидация frontmatter всех скиллов
python .github/scripts/validate_skills.py

# 2. Проверка markdown-ссылок
npx -y markdown-link-check README.md

# 3. Линтинг YAML
yamllint -c .github/yamllint.yml templates/ benchmarks/ skills/
```

Порядок: skill validation → links → YAML. Любая ошибка — PR не пройдёт.

## Структура репозитория

| Папка | Назначение |
|-------|------------|
| `skills/*/*/SKILL.md` | Устанавливаемые скиллы (симлинк в `~/.hermes/skills/`) |
| `templates/config/` | Шаблоны конфигов: minimum, telegram-bot, production, cost-optimized, security-hardened |
| `templates/compose/` | Docker compose для Langfuse стека |
| `templates/systemd/` | Systemd service-файлы |
| `templates/cron/` | Production cron schedule |
| `part*.md` | Основной гайд (24 части) |
| `diagrams/` | Mermaid диаграммы архитектуры |
| `benchmarks/` | Тесты производительности (matrix.yaml + README) |
| `docs/wizard/` | Интерактивный конфиг-визард |

## Требования к SKILL.md frontmatter

```yaml
---
name: <slug>
description: <минимум 10 символов>
when_to_use:
  - <триггер1>
  - <триггер2>
toolsets:
  - terminal
  - github
---
```

Допустимые toolsets: `terminal`, `file`, `github`, `delegate_task`, `classify`, `telegram`, `web`, `browser`, `email`, `discord`, `slack`, `memory`.

## Конвенции коммитов

- **Префиксы PR**: `docs:`, `skill:`, `template:`, `bench:`, `fix:`
- **Кросс-ссылки**: относительные пути (`./partN-foo.md`) — работают в GitHub, VSCode и статике
- **Без секретов**: в примерах использовать `${VAR}` вместо реальных ключей
- **Шаблоны**: комментировать неочевидные поля + добавить заголовок с назначением

## Ссылки

- [CONTRIBUTING.md](./CONTRIBUTING.md) — полная структура и требования
- [skills/README.md](./skills/README.md) — каталог скиллов
- [.github/workflows/ci.yml](./.github/workflows/ci.yml) — что проверяет CI