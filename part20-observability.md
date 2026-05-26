# Часть 20: Наблюдаемость (Observability) и контроль расходов — Langfuse, Helicone, Kanban, /usage, сценарии маршрутизации (Routing Playbooks)

*Нельзя оптимизировать то, что не видно. Hermes отслеживает токены, задержки (latency) и ошибки нативно, но когда вы работаете через CLI + Telegram + Discord + Google Chat + cron + воркеры (worker) Kanban, вам понадобится настоящий стек трассировки (tracing). Эта часть настраивает Langfuse, Helicone или OpenTelemetry → Phoenix одним блоком конфига, а затем даёт сценарий маршрутизации (routing playbook), который снизил расходы нашего тестового деплоя с $34 до $3 за реализацию фичи.*

---

## Трёхуровневый стек (The Three-Level Stack)

```
┌────────────────────────────────────────────────────────┐
│  Уровень 3 — Хостинговая трассировка (Langfuse / Helicone / Phoenix)│
│  Воспроизводимые трейсы (traces), версионирование промптов, evals    │
└────────────────────────────────────────────────────────┘
                            ↑
┌────────────────────────────────────────────────────────┐
│  Уровень 2 — Внутренности Hermes (/usage, /status, dashboard)│
│  Количество токенов, заголовки rate-limit, стоимость за сессию (session)│
└────────────────────────────────────────────────────────┘
                            ↑
┌────────────────────────────────────────────────────────┐
│  Уровень 1 — Логи (~/.hermes/logs/*, `hermes logs tail`)  │
│  Сытые события, вызовы инструментов (tools), ошибки     │
└────────────────────────────────────────────────────────┘
```

У вас всегда есть Уровни 1 и 2. Уровень 3 — это мультипликатор силы (force multiplier), когда вы тратите более $50/мес на LLM-вызовы.

---

## Уровень 1 + 2 — Что идёт в комплекте с Hermes

### `/usage`

```
/usage                              # Текущая сессия (session)
/usage 7d                           # Скользящее окно в 7 дней
/usage --by-provider                # Детализация по провайдерам (providers)
/usage --by-skill                   # Какие навыки (skills) сжигают токены
/usage --by-gateway                 # CLI vs Telegram vs Discord
```

Начиная с v0.9.0 это теперь включает **заголовки rate-limit**, полученные от каждого провайдера — вы можете видеть «насколько я близок к потолку 5M/мин» без погружения в логи.

### Dashboard Analytics (Аналитика панели управления)

[Веб-панель (Web Dashboard)](./part12-web-dashboard.md) содержит вкладку Analytics (Аналитика) со следующими данными:

- Стоимость за день / неделю / месяц
- Токены на входе и выходе (с учётом потоковой передачи — streaming)
- Использование по навыкам (какие из них действительно оправдывают свою стоимость токенов)
- Распределение вызовов инструментов (действительно ли вы используете все эти MCP?)
- Доля ошибок по провайдерам (для настройки failover)

### `hermes logs`

```bash
hermes logs tail -f                 # Живая лента (live tail), все шлюзы (gateways)
hermes logs search "TokenLimit"     # Поиск (grep)
hermes logs export --since 7d       # Экспорт в JSONL для офлайн-анализа
```

Комбинируйте с `jq` или загружайте в DuckDB для ad-hoc-анализа стоимости:

```bash
hermes logs export --since 30d --format jsonl \
  | duckdb -c "SELECT gateway, SUM(tokens_out) FROM read_json_auto('/dev/stdin') GROUP BY 1 ORDER BY 2 DESC"
```

---

## Уровень 3 — Langfuse (рекомендуемый вариант по умолчанию)

Langfuse — это опция «всё в одном»: трассировка (tracing), управление промптами, оценки (evals), возможность самостоятельного хостинга (self-hostable). Если вы не знаете, с чего начать, начните здесь. Начиная с v0.12, Langfuse также поставляется как встроенный плагин (plugin) наблюдаемости, поэтому предпочтительнее включать его, а не использовать самодельные хуки.

```bash
hermes plugins enable observability/langfuse
```

### Настройка (облачный хостинг — Hosted Cloud)

```yaml
# ~/.hermes/config.yaml
observability:
  langfuse:
    enabled: true
    host: https://cloud.langfuse.com
    public_key: ${LANGFUSE_PUBLIC_KEY}
    secret_key: ${LANGFUSE_SECRET_KEY}
    sample_rate: 1.0                # Уменьшите при очень высоком объёме
    traced_tools:                    # Какие вызовы инструментов (tools) захватывать
      - terminal
      - github
      - claude-code
      - gemini-cli
    redact_payloads: true            # Редактирует перед отправкой (соответствует вашим security.secrets.patterns)
```

Получите ключи на https://cloud.langfuse.com → Settings → API Keys. Бесплатный тариф покрывает большинство индивидуальных пользователей.

### Самостоятельный хостинг Langfuse (Self-Hosted Langfuse)

Для конфиденциальности или соответствия требованиям — одна команда на VPS с Docker:

```bash
curl -fsSL https://langfuse.com/docker-compose.yml -o langfuse.yml
docker compose -f langfuse.yml up -d
```

Укажите `host:` на свой домен. Hermes отправляет OTLP через HTTPS, так что Caddy с Let's Encrypt работает без проблем.

### Что вы увидите

Каждый шаг (turn) Hermes становится трейсом (trace). Каждый трейс содержит спаны (spans) для:

- `agent.turn` (корневой — root)
  - `llm.call` (с промптом, завершением, токенами, стоимостью, задержкой)
  - `tool.call` (каждый инструмент с аргументами, результатом, длительностью)
    - вложенный `llm.call` для MCP-серверов с включённым сэмплингом
  - `memory.search` (запросы и совпадения)
  - `skill.load` (какие навыки (skills) были подтянуты)
  - `kanban.task` / `kanban.worker` когда постоянная полоса (board lane) доски захватывает или завершает работу

Воспроизведите любой шаг, проверьте точный промпт, сравните с предыдущими запусками, оцените завершения по датасетам (datasets). Так вы найдёте шаг, который потратил $4 на «как назвать эту переменную».

---

## Уровень 3 — Helicone (сначала шлюз, ноль кода — Gateway-First, Zero Code)

Helicone — это опция «замените базовый URL и работайте». Вы не добавляете SDK трассировки — вы маршрутизируете свой LLM-трафик через прокси, который его наблюдает.

```yaml
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
    base_url: https://anthropic.helicone.ai
    headers:
      Helicone-Auth: Bearer ${HELICONE_API_KEY}
      Helicone-Property-Session: ${HERMES_SESSION_ID}
      Helicone-Property-Skill: ${HERMES_ACTIVE_SKILL}

  openai:
    api_key: ${OPENAI_API_KEY}
    base_url: https://oai.helicone.ai/v1
    headers:
      Helicone-Auth: Bearer ${HELICONE_API_KEY}
      Helicone-Cache-Enabled: "true"   # Автоматическое кэширование промптов
```

Hermes передаёт ID сессии (session) и имя навыка (skill) как пользовательские свойства Helicone, так что вы можете фильтровать трейсы по навыку/сессии в интерфейсе Helicone. Попадания в кэш (идентичные промпты) бесплатны — это само по себе заметно сокращает счета для повторяющихся навыков.

Выбирайте Helicone вместо Langfuse, когда:

- Вы хотите интеграцию без написания кода
- Вы хотите бесплатное кэширование промптов на уровне провайдера
- Вас в основном интересуют панели стоимости и задержки, а не управление промптами

---

## Уровень 3 — OpenTelemetry → Phoenix (в первую очередь стандарты — Standards-First)

Если вы уже используете OpenTelemetry (Grafana, Datadog, Honeycomb), подключите Hermes к своему существующему конвейеру:

```yaml
observability:
  otel:
    enabled: true
    endpoint: https://otel.yourdomain.com:4318
    protocol: http/protobuf
    headers:
      authorization: Bearer ${OTEL_TOKEN}
    attributes:
      service.name: hermes-prod
      deployment.environment: production
```

Hermes отправляет спаны (spans) `gen_ai.*` в соответствии с соглашениями [OpenInference](https://github.com/Arize-ai/openinference). Направьте их в [Arize Phoenix](https://phoenix.arize.com) (самостоятельный хостинг или облако) для LLM-специфичного обзора; или в ваш существующий Grafana/Tempo для обзора «единое окно» (one pane of glass).

---

## Сценарий маршрутизации стоимости (Cost Routing Playbook — тот самый, который реально экономит деньги)

### Правило 1: Маршрутизация по сложности задачи, а не по умолчанию

Основная причина раздувания стоимости Hermes — использование самой дорогой frontier-модели для задач, с которыми Gemini Flash, Kimi/Moonshot, GLM, MiniMax, Cerebras или локальная модель справились бы не хуже. Настройте **умолчание с учётом задачи (task-aware default)**:

```yaml
model_routing:
  default:
    model: claude-sonnet
    provider: anthropic
  routes:
    - match: { intent: [classification, extraction, triage, sum_under_500_tokens] }
      model: gemini-3.1-flash
      provider: google
    - match: { intent: long_context, tokens_gte: 150000 }
      model: gemini-3.1-pro
      provider: openrouter
    - match: { intent: [write_code, refactor, debug], complexity: medium }
      model: glm
      provider: zai
    - match: { intent: [write_code, refactor, debug], complexity: high }
      model: claude-sonnet
      provider: anthropic
    - match: { intent: [reasoning, math], complexity: high }
      model: reasoning
      provider: openai
```

Hermes классифицирует намерение (intent) с помощью крошечного промпта (~100 токенов) и маршрутизирует соответствующим образом. Эмпирически:

| Сценарий | Наивное frontier-умолчание | С маршрутизацией | Экономия |
|----------|----------------------------|--------|---------|
| Реализация фичи (100 вызовов) | ~$34 | ~$3 (в основном Kimi/GLM) | 91% |
| Суммаризация длинного документа (10 вызовов, 200K каждый) | ~$42 | ~$4 (Gemini Pro) | 90% |
| Ежедневная triage-классификация | ~$18/день | ~$1/день (Flash) | 94% |

### Правило 2: Кэширование промптов — бесплатные деньги

Каждый стабильный блок (системный промпт, навык, SOUL.md, дайджест памяти (memory)) должен кэшироваться:

```yaml
prompt_caching:
  enabled: true
  providers: [anthropic, openai, helicone]
  cache_system_prompt: true          # Самый большой выигрыш
  cache_skills: true
  cache_memory_digest: true
  min_cache_tokens: 1024             # Минимум Anthropic
```

Скидка Anthropic на кэширование промптов составляет ~90% на кэшированных чтениях. Для системного промпта на 5K токенов, используемого 100 раз в день, это реальная экономия $2–5 в день.

### Правило 3: Используйте Fast Mode (Быстрый режим) хирургически

[Fast Mode (Быстрый режим)](./part14-fast-mode-watchers.md) (`/fast`) стоит дороже за токен, но снижает задержку в очереди. Используйте его для:

- Интерактивных CLI-сессий, где вы следите за выводом
- Telegram-бесед, где пользователь ждёт ответа
- Голосовых потоков в реальном времени

Не используйте для:

- Cron / запланированных задач
- Ночных заданий анализа
- Длительных массовых операций

```yaml
fast_mode:
  defaults:
    cli: on
    telegram: on
    discord: on
    cron: off
    webhooks: off
  user_override: true                # Пользователь может переключить командой /fast
```

### Правило 4: Контекст — это реальная стоимость — используйте `/compress`

100-й шаг большинства сессий стоит в 10 раз дороже 10-го шага. [`/compress <topic>`](./part14-fast-mode-watchers.md#compress-topic--guided-compression) вместе с подключаемым контекстным движком может ограничить стоимость за шаг:

```yaml
compression:
  auto:
    enabled: true
    at_tokens: 48000                 # Сжимать, когда сессия превышает этот порог
    preserve:
      - last_n_turns: 10
      - tool_results_matching: "error|ERROR|failed"
    topics_from: active_skill         # Использовать имя активного навыка (skill) как тему сжатия
```

### Правило 5: Оповещения об аномалиях стоимости

```yaml
alerts:
  cost_spike:
    window: 1h
    threshold_usd: 5                 # Оповещение, если > $5 за час
    channel: telegram_private
  token_anomaly:
    window: 10m
    threshold_tokens_per_turn: 30000
    channel: telegram_private
```

Ловит зацикленные циклы (навык, застрявший в торнадо повторных попыток — retry tornado) и попытки инъекции промптов (атакующий пытается сжечь ваши токены).

---

## Предотвращение регрессий с помощью оценок (Eval-Driven Regression Prevention)

Как только у вас есть Langfuse, добавьте датасет (dataset) и оценки (evals) для критических путей:

```bash
# Одноразовая настройка
hermes evals init
hermes evals dataset create telegram-support-flows
hermes evals dataset add telegram-support-flows ~/.hermes/traces/support/*.json

# Запускать при каждом релизе
hermes evals run telegram-support-flows --model anthropic/claude-sonnet
hermes evals run telegram-support-flows --model zai/glm     # Проверить, проходит ли более дешёвая модель
hermes evals compare
```

Именно так вы уверенно заменяете модель за $10/Mtok на модель за $0.30/Mtok — эмпирически, а не по ощущениям.

---

## Что дальше

- [Часть 19: Сценарий безопасности (Security Playbook)](./part19-security-playbook.md) — настройте оповещения о стоимости как сигнал обнаружения инъекций
- [Часть 17: MCP-серверы](./part17-mcp-servers.md) — затраты на сэмплинг MCP также отображаются в трейсах (traces)
- [Часть 14: Быстрый режим (Fast Mode)](./part14-fast-mode-watchers.md) — переключатель быстрого режима, упомянутый выше
- [Часть 6: Сжатие контекста (Context Compression)](./part6-context-compression.md) — система сжатия, лежащая в основе Правила 4
