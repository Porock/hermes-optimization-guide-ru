# Экосистема (ecosystem) Hermes

Канонический справочник «где найти X для Hermes». Ведётся параллельно с гайдом — если вы выпустили что-то полезное, откройте PR, чтобы добавить это.

---

## Серверы MCP, которые стоит установить

### Официальные / эталонные
- [`@modelcontextprotocol/server-github`](https://www.npmjs.com/package/@modelcontextprotocol/server-github) — PR, issues, поиск кода, Actions
- [`@modelcontextprotocol/server-filesystem`](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) — чтение/запись в ограниченные директории
- [`@modelcontextprotocol/server-postgres`](https://www.npmjs.com/package/@modelcontextprotocol/server-postgres) — SQL только на чтение
- [`@modelcontextprotocol/server-sqlite`](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sqlite) — локальная SQLite
- [`@modelcontextprotocol/server-puppeteer`](https://www.npmjs.com/package/@modelcontextprotocol/server-puppeteer) — автоматизация headless-браузера
- [`@modelcontextprotocol/server-memory`](https://github.com/modelcontextprotocol/servers/tree/main/src/memory) — лёгкая KV-память (memory)
- [`@modelcontextprotocol/server-google-drive`](https://www.npmjs.com/package/@modelcontextprotocol/server-gdrive) — чтение Google Drive

### Проприетарные MCP от вендоров
- [`AWS Labs MCP servers`](https://github.com/awslabs/mcp) — AWS docs, CDK, стоимость, диаграммы и вспомогательные утилиты для сервисов
- [`@cloudflare/mcp-server-cloudflare`](https://github.com/cloudflare/mcp-server-cloudflare) — Workers, KV, D1, R2
- [`@supabase/mcp-server-supabase`](https://github.com/supabase-community/supabase-mcp/tree/main/packages/mcp-server-supabase) — Postgres + хранилище + аутентификация
- [`@stripe/mcp-server-stripe`](https://github.com/stripe/ai/tree/main/tools/modelcontextprotocol) — чтение платежей + ограниченная запись
- [`Linear remote MCP`](https://linear.app/docs/mcp) — отслеживание задач
- [`@notionhq/notion-mcp-server`](https://github.com/makenotion/notion-mcp-server) — чтение/запись страниц
- [`@browserbase/mcp-server`](https://github.com/browserbase/mcp-server-browserbase) — управляемый headless-браузер
- [`@chromadb/mcp-server-chroma`](https://github.com/chroma-core/chroma-mcp) — векторный поиск

### Сообщество
- [`Mem0 remote MCP`](https://docs.mem0.ai/platform/mem0-mcp) — постоянная кросс-устройственная память
- [`arxiv-mcp-server`](https://github.com/blazickjp/arxiv-mcp-server) — поиск по arxiv + извлечение PDF
- [`mcp-server-atlassian`](https://github.com/sooperset/mcp-atlassian) — Jira + Confluence
- [`@modelcontextprotocol/server-slack`](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/slack) — сообщения, поиск, профиль
- [`dbt-mcp`](https://github.com/dbt-labs/dbt-mcp) — dbt Cloud
- [`e2b-dev/mcp-server`](https://github.com/e2b-dev/mcp-server) — одноразовые Python-песочницы
- [`mcp-obsidian`](https://github.com/MarkusPfundstein/mcp-obsidian) — ваше хранилище Obsidian

См. [Часть 17](./part17-mcp-servers.md) с описанием шаблонов установки и модели доверия.

---

## Интеграции (integration) с агентами (agent) кодинга

- [Claude Code](https://docs.claude.com/en/docs/claude-code) — `claude -p` + ACP; лучший автоматический PR-трек с Sonnet 5 / Opus 4.7
- [OpenAI Codex CLI](https://github.com/openai/codex) — `codex -p`; мощный трек исправления багов в песочнице с GPT-5.5/Codex models
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) — `gemini -p` (бесплатный тариф через OAuth); лучший трек чтения/исследования репозитория
- [OpenCode](https://github.com/sst/opencode) — мультимодельный оркестратор; полезен с бюджетными треками Kimi K2.6 / GLM
- [Aider](https://aider.chat) — REPL для парного программирования

См. [Часть 18](./part18-coding-agents.md) и [Часть 23](./part23-tenacity-stack.md#2-add-worker-lanes-instead-of-giant-prompt-swarms).

---

## Плагины (plugin) дашборда

- `hermes-dashboard-lightrag` — вкладка обозревателя графов
- `hermes-dashboard-langfuse` — встроенные трейсы Langfuse для текущей сессии (session)
- `hermes-dashboard-costs` — график затрат по провайдерам (provider) / навыкам (skill)

(Поддерживаются сообществом; см. [Часть 12](./part12-web-dashboard.md#dashboard-plugins).)

---

## Наблюдаемость + стоимость

- [Langfuse](https://github.com/langfuse/langfuse) — самостоятельный хостинг: трассировка + промпты + оценки
- [Helicone](https://github.com/Helicone/helicone) — прокси со шлюзом (gateway) в основе, автоматическое кэширование
- [Arize Phoenix](https://github.com/Arize-ai/phoenix) — нативный OpenTelemetry, офлайн
- [OpenRouter](https://openrouter.ai) — агрегатор провайдеров с маршрутизацией по стоимости
- [Сравнение цен Helicone](https://www.helicone.ai/llm-cost) — текущие розничные цены
- [Artificial Analysis](https://artificialanalysis.ai) — сторонние бенчмарки

См. [Часть 20](./part20-observability.md).

---

## Исследования безопасности / заслуживающие внимания CVE (2026)

- **Comment and Control (2026-04-15)** — кросс-вендорная инъекция промптов через заголовки PR на GitHub, затрагивающая Claude Code, Gemini CLI, GitHub Copilot Agent. См. защитную статью, указанную в [Части 19](./part19-security-playbook.md).
- **Отравление stdio в MCP** — недоверенные npm-пакеты, проксирующие stdio-трафик MCP. Смягчается фиксацией версий + аудитами Socket.dev/Semgrep.
- **Атаки повторного воспроизведения вебхуков** — напоминание, что только HMAC + TTL вместе, а не один HMAC, предотвращают повторное воспроизведение.

См. [Часть 19](./part19-security-playbook.md).

---

## Шаблоны в этом репозитории

- [`templates/config/*`](./templates/config/) — пять базовых конфигураций с учётом лучших практик
- [`templates/compose/langfuse-stack.yml`](./templates/compose/langfuse-stack.yml) — самостоятельный хостинг Langfuse v3
- [`templates/caddy/Caddyfile`](./templates/caddy/Caddyfile) — обратный прокси + автоматический TLS
- [`templates/systemd/hermes.service`](./templates/systemd/hermes.service) — усиленный unit-файл
- [`scripts/vps-bootstrap.sh`](./scripts/vps-bootstrap.sh) — от свежего VPS до продакшена за один запуск

---

## В других местах в интернете

- [Hermes Agent (Nous Research)](https://github.com/NousResearch/hermes-agent) — вышестоящий проект
- [Model Context Protocol](https://modelcontextprotocol.io) — спецификация + каталог серверов
- [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)
- [Discord Nous Research](https://discord.gg/nousresearch) — поддержка сообщества

---

## Отправить запись

Откройте PR с добавлением в соответствующий раздел. Требования:
1. Ссылка на реальный публичный репозиторий
2. Описание в одну строку того, что он делает
3. (Для серверов MCP) лицензия + рекомендация по уровню доверия

См. [CONTRIBUTING.md](./CONTRIBUTING.md).
