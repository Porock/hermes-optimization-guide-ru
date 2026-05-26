# Часть 17: MCP-серверы (MCP Servers) — Дайте Hermes Любой Инструмент Без Единой Строки Прокси-Кода

*Model Context Protocol (MCP) — это «USB-C для AI-агентов (agents)»: стандартный способ подключения любого инструментального сервера (server) к любому агенту. Hermes поддерживает MCP нативно начиная с [v0.7.0](https://github.com/NousResearch/hermes-agent/releases/tag/v2026.4.3). Это та часть руководства, которую никто не читает, пока не осознаёт, что может перестать писать адаптеры для инструментов (tools) вручную.*

---

## Почему Это Важно

До появления MCP каждый фреймворк агентов имел свою собственную схему вызова инструментов. Вы писали GitHub-инструмент для Hermes, затем переписывали его для Claude Code, затем снова переписывали для Cursor. И все три вызывали один и тот же GitHub API.

MCP (представленный Anthropic, теперь де-факто стандарт для Claude Code, Cursor, GitHub Copilot, Devin и Hermes) определяет:

- **Обнаружение инструментов (tool discovery)** — стандартный JSON-формат для описания входных и выходных данных
- **Транспорты (transports)** — stdio (локальный подпроцесс) и HTTP (удалённый сервер)
- **Двунаправленная семплинг (bi-directional sampling)** — MCP-серверы могут попросить агента выполнить LLM-вызов от их имени

Hermes встраивается в эту экосистему. Укажите ему любой MCP-сервер — созданный сообществом или ваш собственный — и инструменты появятся рядом со встроенными инструментами Hermes без изменений в коде. Это самый эффективный час, который вы потратите на оптимизацию своего агента.

---

## Как MCP Встраивается в Hermes

```
┌────────────────────────────────────────────────────┐
│  Hermes Agent                                       │
│  ┌──────────────────────────────────────────────┐  │
│  │  Встроенные инструменты (терминал, навыки,   │  │
│  │  память)                                      │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │  MCP-клиент (MCP Client)                     │  │
│  │  ├─ github-mcp     (stdio, подпроцесс)       │  │
│  │  ├─ postgres-mcp   (stdio, подпроцесс)       │  │
│  │  ├─ mem0-mcp       (http, удалённый)         │  │
│  │  └─ your-mcp       (stdio или http)          │  │
│  └──────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────┘
```

Hermes автоматически обнаруживает инструменты при запуске и подписывается на динамические обновления — если MCP-сервер добавляет новый инструмент в середине сессии (session), Hermes подхватывает его без перезапуска.

---

## Конфигурация

MCP-серверы указываются в ключе `mcp_servers` в файле `~/.hermes/config.yaml`.

### stdio-серверы (Локальный Подпроцесс)

```yaml
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}

  filesystem:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/home/you/projects"]

  postgres:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"]
```

Hermes запускает подпроцесс при старте, передаёт JSON-RPC через stdio и завершает его при выходе. Перезапустите Hermes после добавления нового stdio-сервера.

### HTTP / SSE-серверы (Удалённые)

```yaml
mcp_servers:
  mem0:
    url: https://mcp.mem0.ai/sse
    headers:
      Authorization: Bearer ${MEM0_API_KEY}

  cloudflare:
    url: https://observability.mcp.cloudflare.com/sse
    headers:
      Authorization: Bearer ${CLOUDFLARE_API_TOKEN}
```

HTTP-серверы могут добавлять/удалять инструменты на лету. Hermes обрабатывает переподключение с экспоненциальной задержкой (exponential backoff).

### Ограничение Области Видимости

Некоторые серверы болтливы — вы не хотите загружать все их инструменты в каждый разговор. Ограничьте их область:

```yaml
mcp_servers:
  postgres:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"]
    enabled_for:                     # Загружать только в этих сессиях
      - profile: engineering
      - channel: "#data-questions"
    tools_allowlist:                 # Предоставлять только эти инструменты
      - query
      - describe_table
```

Без `tools_allowlist` будут доступны все инструменты, которые предоставляет сервер.

---

## MCP-Серверы, Которые Стоит Установить Сегодня

Эти окупаются в течение дня:

> **Реалии 2026 года:** MCP также является границей цепочки поставок. Предпочитайте официальные серверы, фиксируйте версии пакетов, ограничивайте корневые каталоги файловой системы и держите `allow_sampling: false`, если серверу действительно не нужно вызывать LLM.

| Сервер | Что добавляет | Зачем он вам |
|--------|--------------|--------------|
| **@modelcontextprotocol/server-github** | Issues, PR, поиск по репозиторию, диффы веток | Hermes становится осведомлённым о коде коллегой |
| **@modelcontextprotocol/server-filesystem** | Ограниченные чтение/запись/поиск файлов | Безопаснее, чем давать доступ к терминалу |
| **@modelcontextprotocol/server-postgres** | SQL только для чтения | Ответить «что в базе?» без раскрытия DSN |
| **@modelcontextprotocol/server-sqlite** | Локальный анализ SQLite | Отлично для логов, аналитических срезов |
| **@modelcontextprotocol/server-puppeteer** | Автоматизация браузера | Дополнение к Browser Use в шлюзе (gateway) инструментов; изолируйте его строго |
| **@modelcontextprotocol/server-memory** | Графовая память (knowledge-graph memory) | Сочетается с [Частью 3 LightRAG](./part3-lightrag-setup.md) для избыточности |
| **mcp.mem0.ai** | Облачная долговременная память | Кросс-платформенная память между Hermes и Claude Code |
| **Cloudflare Observability MCP** | Запрос логов/аналитики Workers | Если вы используете Cloudflare |
| **@supabase/mcp-server-supabase** | Supabase RPC + Postgres + хранилище | Одна конфигурация для всего бэкенда |
| **linear-mcp** | CRUD задач Linear | Превратите Hermes в исполнителя задач |
| **stripe-mcp** | Чтение Stripe (клиенты, подписки) | Триаж поддержки из Telegram |
| **@notionhq/notion-mcp-server** | Страницы и базы данных Notion | Вики компании как обоснованный контекст |
| **@browserbase/mcp** | Головной браузер как услуга | Парсинг сайтов, с которыми не справляется Firecrawl |
| **@chroma-core/chroma-mcp** | Векторы ChromaDB | Работает вместе с LightRAG |

Полный каталог — в [MCP Registry](https://registry.modelcontextprotocol.io/) и в списке `awesome-mcp-servers` на GitHub.

---

## Создание Собственного MCP-Сервера (Быстро)

Минимальный Node MCP-сервер занимает ~30 строк. Python — аналогично. Укажите его Hermes как любой другой stdio-сервер.

```javascript
// my-mcp/index.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "my-mcp", version: "0.1.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "deploy_staging",
    description: "Deploys current git HEAD to the staging environment",
    inputSchema: {
      type: "object",
      properties: { service: { type: "string" } },
      required: ["service"]
    }
  }]
}));

server.setRequestHandler("tools/call", async (req) => {
  if (req.params.name === "deploy_staging") {
    const result = await deployStaging(req.params.arguments.service);
    return { content: [{ type: "text", text: result }] };
  }
});

await server.connect(new StdioServerTransport());
```

Зарегистрируйте его:

```yaml
mcp_servers:
  ops:
    command: node
    args: ["/home/you/mcp/my-mcp/index.js"]
```

Теперь `deploy_staging` — это инструмент, который Hermes может вызывать из любого интерфейса — CLI, Telegram, iMessage, Discord — без изменения кода Hermes.

---

## Семплинг (Sampling): MCP-Сервер Вызывает LLM

Это убийственная функция MCP и причина, по которой он важен именно для агентов. MCP-серверы могут запрашивать LLM-инференс у Hermes через `sampling/createMessage`:

- Парсерный MCP загружает неструктурированную страницу → просит LLM Hermes извлечь структурированные данные → возвращает структурированные данные агенту.
- MCP проверки безопасности читает дифф → просит LLM классифицировать серьёзность → возвращает метку триажа.
- MCP перевода читает файл → просит LLM локализовать его → записывает результат.

Hermes обрабатывает запрос инференса активным провайдером (provider) и учитывает токены в текущей сессии. Включите семплинг для сервера:

```yaml
mcp_servers:
  scraper:
    command: node
    args: ["./scraper-mcp.js"]
    allow_sampling: true              # По умолчанию выключено
    sampling_model: gpt-5-mini        # Опционально: закрепить более дешёвую модель для семплинга
```

**Замечание по безопасности:** Семплинг означает, что MCP-сервер может расходовать ваши токены. Включайте его только для серверов, которым доверяете. См. [Часть 19](./part19-security-playbook.md#mcp-server-trust-model).

---

## Наблюдение за MCP-Трафиком

```bash
/mcp list                            # Показать зарегистрированные серверы + количество инструментов
/mcp reload                          # Перезагрузить серверы без перезапуска Hermes
/mcp disable github                  # Временно отключить
/mcp enable github                   # Включить обратно
```

[Веб-панель (Web Dashboard)](./part12-web-dashboard.md) имеет вкладку **MCP Servers**, которая показывает статус подключения, список инструментов, последние вызовы и логи ошибок для каждого сервера. Это самый быстрый способ отладки проблемного MCP.

Установите `HERMES_MCP_LOG=debug` в вашем `.env`, чтобы получить полные JSON-RPC трассы в `~/.hermes/logs/mcp.log`. Отключайте это в продакшене — трассы содержат аргументы и результаты инструментов.

---

## Когда MCP Избыточен

MCP добавляет процесс (или сетевой прыжок) на каждый инструмент. Для того, что уже есть внутри Hermes, не стоит:

- **Команды терминала** — просто используйте встроенный инструмент `terminal`.
- **Редактирование файлов** — встроенные файловые инструменты быстрее, чем filesystem MCP, если файлы локальны.
- **Навыки (skills)** — если рабочий процесс детерминирован, [навык](./part5-creating-skills.md) дешевле поддерживать.

Используйте MCP, когда вам нужно:
- Инструмент, у которого уже есть поддерживаемый сообществом сервер (GitHub, Slack, Postgres и т.д.)
- Инструмент, которым вы хотите делиться с другими агентами (Claude Code, Cursor, Copilot)
- Инструмент, которому нужна своя среда выполнения (Node/Go/Rust), которую вы не хотите встраивать в Hermes

---

## Устранение Неполадок

| Симптом | Вероятная причина | Исправление |
|---------|--------------|-----|
| `MCP server 'github' failed to start` | `npx` не найден в PATH окружения шлюза | Используйте абсолютный путь в `command:` или укажите `PATH` в `env:` |
| Сервер показывает подключение, но 0 инструментов | Разрешения — в переменных окружения сервера отсутствует токен аутентификации | Проверьте записи `env:` и что указанные `${VARS}` существуют в `.env` |
| Инструменты видны в CLI, но не в Telegram | Процесс шлюза имеет своё окружение — перезапустите его после изменения конфига | `hermes gateway restart` |
| Постоянные переподключения на HTTP-сервере | Тайм-аут SSE за обратным прокси | Установите `proxy_read_timeout 300s` в nginx/Caddy |
| `sampling not permitted` в логах сервера | `allow_sampling: false` (по умолчанию) | Установите `allow_sampling: true` в блоке сервера |

---

## Что Далее

- [Часть 18: Делегирование Кодирующим Агентам](./part18-coding-agents.md) — используйте Claude Code, Codex и Gemini CLI как сабагентов (subagents), вызываемых через Hermes (некоторые также поставляются с MCP-серверами)
- [Часть 19: Практическое Руководство по Безопасности](./part19-security-playbook.md) — модель доверия MCP, лимиты семплинга и как недоверенные MCP изолируются
- [Часть 12: Веб-Панель](./part12-web-dashboard.md) — панель MCP Servers

(Конец файла)
