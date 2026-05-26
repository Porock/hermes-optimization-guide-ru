# Руководство по оптимизации Hermes (на русском языке)

> [English полная версия](./README.md) · Это страница-введение, основной текст на русском языке.

Практическое руководство + готовые к использованию артефакты (навыки (skills), шаблоны конфигов, скрипты инфраструктуры) для [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) (текущая версия — v0.13.0).

## Быстрый старт (одна команда)

```bash
# На новом Debian 12 / Ubuntu 24.04 (VPS Hetzner CX22 ~5$/мес)
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | sudo bash
```

Или см. [docs/quickstart.md](./docs/quickstart.md) — за 5 минут до Telegram-бота.

## Обзор содержимого

- **24 части руководства** (в README + отдельные файлы part6–part23) — Kanban, `/goal`, Checkpoints v2, Curator, TUI, плагины, LightRAG, Telegram, MCP, безопасность, observability, удалённые песочницы
- **13 устанавливаемых навыков (skills)** (`skills/`) — аудит, бэкапы, проверка зависимостей, отчёт о затратах, классификация Telegram, PR-ревью, сортировка входящих, еженедельник Hermes, фильтрация спама, подготовка к совещаниям
- **5 шаблонов конфигов для продакшена** (`templates/config/`) — minimum / telegram-bot / production / cost-optimized / security-hardened
- **Инфраструктура** (`templates/compose/`, `templates/caddy/`, `templates/systemd/`, `scripts/`) — self-hosted Langfuse, обратный прокси Caddy, закалённые systemd-юниты, скрипт установки на VPS
- **Mermaid-диаграммы архитектуры** (`diagrams/`)
- **Воспроизводимые бенчмарки** (`benchmarks/`) — 12 моделей × 5 задач, с методологией
- **Экосистемный справочник** ([ECOSYSTEM.md](./ECOSYSTEM.md)) — MCP-серверы, кодирующие агенты, плагины дашборда
- **Интерактивный мастер конфигурации** ([docs/wizard/](./docs/wizard/)) — генерация `config.yaml` в браузере

## Рекомендуемый порядок чтения

1. Хотите быстро запустить Telegram-бота → [docs/quickstart.md](./docs/quickstart.md)
2. Хотите понять архитектуру → [diagrams/architecture.md](./diagrams/architecture.md)
3. Хотите сэкономить → раздел "Cost-routing playbook" в [part20-observability.md](./part20-observability.md)
4. Готовитесь к продакшену → выберите подходящий вариант в [docs/reference-architectures/](./docs/reference-architectures/)
5. Публичный деплой → обязательно к прочтению [part19-security-playbook.md](./part19-security-playbook.md)

## Лицензия и участие

MIT. Приветствуются Issue и Pull Requests, см. [CONTRIBUTING.md](./CONTRIBUTING.md).

---

*Перевод на русский язык выполнен энтузиастами сообщества. Оригинальная версия: [English](https://github.com/OnlyTerp/hermes-optimization-guide).*