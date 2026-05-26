# Дорожная карта (Roadmap)

Что появится в следующих версиях. PR приветствуются.

## В процессе

- [ ] **Документация на GitHub Pages** — Astro Starlight с полнотекстовым поиском по всем частям и навыкам (skills)
- [ ] **Asciinema запись** — 60-секундная запись «от нуля до рабочего Telegram бота», встроенная в README
- [ ] **Langfuse dashboard JSON** — импортируемая готовая панель мониторинга для трейсов (traces) Hermes
- [ ] **Upstream PR** в README `NousResearch/hermes-agent` — добавить раздел Community Guides (черновик в `docs/outreach/nous-upstream-pr-body.md`)

## В очереди

- [ ] **Шаблоны навыков (Skill templates)** — генератор заготовок `hermes skills new <name>`
- [ ] **Проверка перекрёстных ссылок (Cross-link checker)** — CI проверка, которая падает, если любая ссылка `[...](./...)` выдаёт 404 (частично: markdown-link-check на изменённых файлах уже работает)
- [ ] **Лента CVE безопасности (Security CVE feed)** — `.github/workflows/cve-watch.yml`, отслеживающий OSV на предмет соответствующих уведомлений
- [ ] **Проход со скриншотами панелей (Dashboard screenshots pass)** — встроить реальные скриншоты в части 12 / 17 / 20

## На рассмотрении

- Нативный пакет навыков (skill pack) Hermes, устанавливаемый через `hermes skills install onlyterp/hermes-optimization-guide`
- Git-теги на каждый релиз, чтобы пользователи могли зафиксироваться на известном рабочем состоянии
- Инкубатор MCP-серверов сообщества — небольшой репозиторий, который выпускает серверы после достижения планки качества

## Готово (недавнее)

- ✅ 2026-05-14 — Обновление v0.13: Kanban, `/goal`, Checkpoints v2, Google Chat, cron без агента (agent), плагины провайдеров (provider plugins) и SOTA моделей мая 2026
- ✅ 2026-04-30 — Обновление v0.11/v0.12: куратор (Curator), TUI, плагины, Bedrock/Azure/LM Studio, Teams/Yuanbao/QQBot, Vercel Sandbox, Часть 22
- ✅ 2026-04-17 — Интерактивный мастер конфигурации (config wizard) (`docs/wizard/`)
- ✅ 2026-04-17 — 4 эталонные архитектуры (homelab / solo-dev / small-agency / road-warrior)
- ✅ 2026-04-17 — CI (markdown-link-check + yamllint + валидатор frontmatter навыков)
- ✅ 2026-04-17 — Страницы входа README на китайском и японском языках
- ✅ 2026-04-17 — Черновики для публикаций (tweet, HN, Reddit, upstream PR, blog post)
- ✅ 2026-04-17 — Устанавливаемая библиотека навыков (skill library) + шаблоны (templates) + скрипт начальной настройки (bootstrap script)
- ✅ 2026-04-17 — Части по MCP, кодинг-агенту (coding-agent), безопасности, наблюдаемости (observability), песочнице (sandbox) (17–21)
- ✅ 2026-04-16 — Обновление v0.9 + v0.10 (части 12–16)

## Как предложить дополнения

Откройте issue с меткой `roadmap`. Укажите:

- Что делает дополнение
- Для кого оно
- Оценка трудозатрат (small / medium / large)
- Готовы ли вы написать это самостоятельно
