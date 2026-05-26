# История изменений (Changelog)

Датированный список значимых обновлений руководства. В стиле [Keep a Changelog](https://keepachangelog.com).

## 2026-05-14 — Hermes v0.13.0 Tenacity Refresh

### Добавлено (Added)
- **Часть 23 — Tenacity Stack**: отказоустойчивые Kanban-доски, воркер-ленты (worker lanes), `/goal`, Checkpoints v2, cron без агента (agent), плагины (plugin) провайдеров (provider) и чеклист обновления до v0.13
- Google Chat в Части 15 как 20-я платформа обмена сообщениями
- Руководство по Kanban-лентам воркеров (worker) в Части 18 для оркестрации Codex/Claude/Gemini/OpenCode
- Рекомендации по безопасности по умолчанию v0.13 в Части 19: редактирование (redaction) включено по умолчанию, allowlist ролей Discord в масштабах гильдии, отклонение незнакомцев в WhatsApp, исправления (fix) TOCTOU для OAuth и auth.json

### Изменено (Changed)
- Бейджи README, раздел «Что нового», оглавление, копия архитектуры и таблицы моделей теперь нацелены на Hermes v0.13.0 (v2026.5.7)
- Обновлены рекомендации по моделям/провайдерам (provider) в Части 9 для SOTA мая 2026: Claude Sonnet 5 / Opus 4.7, GPT-5.5, Gemini 3.1, Kimi K2.6, DeepSeek V4, Qwen3.6, плагины (plugin) провайдеров и медиа-маршрутизация
- Часть 12 обновлена для поддержки Kanban/профиля в панели управления (dashboard)
- Часть 14 обновлена для `/goal`
- Часть 16 обновлена для терминологии отладки/редактирования (debug/redaction) v0.13
- Часть 20 обновлена для наблюдаемости (observability) с учётом Kanban
- Шаблоны конфигов, шаблоны cron, бенчмарки (benchmarks), локализованные README, дорожная карта (roadmap), копия для публикации (outreach), настройки визарда (wizard) обновлены для руководства из 24 частей

### Удалено (Removed)
- Фрейминг v0.12 как актуальной версии (release) из верхнеуровневых указаний
- Устаревшие рекомендации по моделям от апреля 2026 года, где замены мая 2026 теперь являются лучшим выбором по умолчанию

## 2026-04-30 — Hermes v0.11/v0.12 Refresh

### Добавлено (Added)
- **Часть 22 — Latest Power Moves**: куратор (curator), привычки управления через TUI, гигиена контекстных файлов, плагины (plugin), вспомогательные модели, цепочки cron и чеклист обновления до v0.12
- Руководство по куратору (curator) в Части 5, включая пробный прогон (dry-run), планирование, поведение закрепления/архивации и отличия от навыков (skills)/памяти (memory)/контекстных файлов
- Поддержка платформ v0.12: QQBot, Tencent Yuanbao и Microsoft Teams как шлюз (gateway) на базе плагина (plugin)
- Заметки об AWS Bedrock, Azure AI Foundry, LM Studio, GMI Cloud, Tencent TokenHub, MiniMax OAuth, Gemini OAuth и удалённом каталоге моделей в Части 9
- Поддержка Vercel Sandbox в Части 21

### Изменено (Changed)
- Раздел README «Что нового» теперь отражает выпущенные версии (release) v0.11.0 и v0.12.0 вместо отслеживания предполагаемых PR после v0.10
- Часть 12 обновлена для вкладок Chat, Models, плагинов (plugin), элементов управления куратором (curator) и требований установки `web,pty`
- Часть 14 обновлена для `/steer`, `/queue`, `/background`, `/busy` и текущей терминологии Fast Mode
- Часть 18 обновлена для сабагентов (subagent) в роли оркестратора и координации файлов
- Часть 19 обновлена с учётом поверхностей угроз MCP/плагин (plugin)/панель управления (dashboard) и инструкций по жёсткой блокировке v0.12
- Часть 20 обновлена для использования встроенного плагина (plugin) наблюдаемости Langfuse и вспомогательной маршрутизации

### Удалено (Removed)
- Устаревший фрейминг «Cooking on main» и заглушка-раскрытие example.com
- Старое требование установки Gemini CLI для Gemini OAuth

## 2026-04-17 — Визард (Wizard) + Эталонные Архитектуры + CI

### Добавлено (Added)
- **`docs/wizard/index.html`** — интерактивный статический конфиг-визард (wizard); 8 вопросов → готовый к использованию `config.yaml`, работает полностью в браузере (совместим с GitHub Pages)
- **`docs/reference-architectures/`** — 4 полных архитектурных蓝图: Homelab, Solo Developer, Small Agency, Road Warrior
- **`docs/outreach/`** — готовые к публикации черновики: тред для запуска в Twitter, пост для Hacker News, пост для r/LocalLLaMA, тело PR для апстрима в `NousResearch/hermes-agent`, полноценная статья для блога
- **4 новых навыка (skill)**: `ops/daily-inbox-triage`, `ops/hermes-weekly`, `security/spam-trap`, `dev/meeting-prep` (всего навыков (skills): 13)
- **CI** — `.github/workflows/ci.yml`: проверка markdown-ссылок (markdown-link-check), yamllint, валидатор frontmatter навыков (validate_skills.py), дополнительная проверка prettier
- **Локализованные README** — [`README-zh.md`](./README-zh.md), [`README-ja.md`](./README-ja.md) (краткие вводные)

### Изменено (Changed)
- README: бейдж навыков (skills) 9→13, ссылки на языки, строки карты репозитория для визарда (wizard) + эталонных архитектур + материалов для публикации, бейдж CI
- `templates/config/*.yaml` — экранированные подстановки переменных окружения `${VAR}` в потоковых отображениях (flow mappings), чтобы каждый шаблон был валидным YAML

## 2026-04-17 — Устанавливаемые Артефакты (Installable Artifacts)

### Добавлено (Added)
- **`skills/`** — 9 исполняемых файлов `SKILL.md` (audit-mcp, rotate-secrets, audit-approval-bypass, nightly-backup, weekly-dep-audit, cost-report, telegram-triage, pr-review, release-notes)
- **`templates/config/`** — 5 целевых конфигов (minimum, telegram-bot, production, cost-optimized, security-hardened)
- **`templates/compose/langfuse-stack.yml`** — самостоятельно размещаемый Langfuse v3 с ClickHouse + MinIO + Redis
- **`templates/caddy/Caddyfile`** — обратный прокси (reverse-proxy) + эталонный авто TLS
- **`templates/systemd/`** — усиленные `hermes.service` + `hermes-dashboard.service`
- **`templates/cron/production-crons.yaml`** — все рекомендованные плановые задачи
- **`scripts/vps-bootstrap.sh`** — чистый Hetzner CX22 → production Hermes за ~10 минут
- **`diagrams/architecture.md`** — 6 Mermaid-диаграмм (верхнеуровневая, MCP, делегирование, синхронизация песочницы (sandbox), наблюдаемость (observability), безопасность)
- **`benchmarks/README.md` + `matrix.yaml`** — воспроизводимая таблица стоимости/задержки для 12 моделей × 5 задач
- **`ECOSYSTEM.md`** — канонический каталог MCP-серверов, кодинг-агентов (agent), плагинов (plugin) панели управления (dashboard), инструментов наблюдаемости (observability)
- **`ROADMAP.md`** — планы на будущее; приглашение к участию
- **`CONTRIBUTING.md`**, **`CHANGELOG.md`**, **`CODE_OF_CONDUCT.md`** — стандартная гигиена репозитория
- **Шаблоны GitHub issue + PR**
- **`docs/quickstart.md`** — 5-минутная инструкция «копировать-вставить» от нуля до рабочего Telegram-бота

### Изменено (Changed)
- README получил бейджи, раздел «Установить всё», встроенную диаграмму архитектуры, перекрёстные ссылки на экосистему/бенчмарки

## 2026-04-17 — 72-часовое исследование (72h Research Sweep) (PR #6, слито)

### Добавлено (Added)
- Часть 17 — MCP-серверы (MCP Servers)
- Часть 18 — Делегирование кодинг-агентам (Coding Agents) (Claude Code, Codex, Gemini CLI, OpenCode, Aider)
- Часть 19 — Учебник по безопасности (Security Playbook) (защита от инъекций промптов «Comment and Control» от 15 апреля)
- Часть 20 — Наблюдаемость и контроль затрат (Observability & Cost Control) (Langfuse, Helicone, Phoenix)
- Часть 21 — Удалённые песочницы и массовая синхронизация файлов (Remote Sandboxes & Bulk File Sync) (#8018)
- README — дерево решений «Выбери свой путь» (Pick Your Path)
- README — раздел «Cooking on `main`» (PR после v0.10)

### Изменено (Changed)
- Часть 9 — Шпаргалка по флагманским моделям, шпаргалка по маршрутизации задач, OAuth Gemini CLI, Gemini TTS
- Добавлены перекрёстные ссылки в частях 3, 5, 8

## 2026-04-16 — Hermes v0.9 + v0.10 refresh (PR #5, слито)

### Добавлено (Added)
- Часть 12 — Веб-панель управления (Web Dashboard) (`hermes dashboard`)
- Часть 13 — Шлюз инструментов Nous (Nous Tool Gateway)
- Часть 14 — Fast Mode + Фоновые наблюдатели (Background Watchers) + подключаемый механизм контекста (context engine)
- Часть 15 — Новые платформы (iMessage, WeChat, Android/Termux) — всего 16 платформ
- Часть 16 — Резервное копирование (Backup) / Импорт (Import) / сборщик `/debug`

### Изменено (Changed)
- Оглавление README расширено с 11 до 17
- Часть 4 — Telegram переосмыслен как «флагманский из 16 шлюзов (gateway)»
- Часть 9 — добавлена матрица нативных адаптеров (native-adapter)

## Ранее (Earlier)

- Начальное руководство из 11 частей: установка, миграция с OpenClaw, LightRAG, Telegram, навыки (skills), сжатие контекста, память (memory), сабагенты (subagent), пользовательские модели, антипаттерны SOUL, восстановление шлюза (gateway).
