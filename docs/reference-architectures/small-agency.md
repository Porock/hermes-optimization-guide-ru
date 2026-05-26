# Эталонная архитектура (Reference Architecture): Small Agency

**2–6 разработчиков, несколько клиентов, изоляция по клиентам.** Одна инсталляция Hermes плохо масштабируется на команду; эта архитектура (architecture) запускает выделенный профиль (profile) на каждого разработчика/клиента и разделяет только слой observability + аудита.

## Для кого это

- Dev-шопы / консалтинговые агентства, работающие с несколькими клиентскими кодовыми базами
- Небольшие продуктовые команды со строгими требованиями разделения ответственности
- Те, кому нужны аудиторские следы, выдерживающие проверку безопасности клиента

## Стоимость

- **Инфраструктура:** ~$25–50/мес (один CX32 или 2× CX22)
- **LLM:** $200–800/мес (с маршрутизацией)
- **Langfuse/observability:** $0 при самостоятельном хостинге или $100+/мес управляемая версия

## Архитектура (Architecture)

```
          Devs (Telegram/Discord DMs, CLI)
             │                          │
             ▼                          ▼
     ┌───────────────────┐    ┌───────────────────┐
     │ Hermes per dev/   │    │ Shared services   │
     │ per client        │    │                   │
     │ (systemd units)   │    │  Langfuse         │
     │                   │    │  Audit log sink   │
     │ hermes@alice.s    │    │  LightRAG (each)  │
     │ hermes@bob.s      │    │  backup target    │
     │ hermes@clientA.s  │    │                   │
     └───────────────────┘    └───────────────────┘
```

- **Шаблонные systemd-юниты** — `hermes@<name>.service`, по одному на разработчика/клиента, каждый со своим `${HOME}/.hermes/` и своим каналом одобрения (DM этого разработчика)
- **LightRAG на каждый экземпляр** — знания клиентов никогда не смешиваются
- **Централизованный Langfuse + журнал аудита** — каждый вызов отслеживается, PID-данные вырезаются на уровне секретов

## Состав

- **1× CX32** (4 vCPU, 8GB RAM) — $12/мес, размещает 3–6 экземпляров Hermes + Langfuse
- **S3/R2 бакет для бэкапов** — зашифрованные еженощные резервные копии (age/gpg)
- **Cloudflare** — DNS + TLS-терминированный обратный прокси (или Caddy, если вы предпочитаете не использовать CF)
- **Linear/Notion/Slack/Google Workspace** — MCP-подключённые, только на чтение для контекста

## Установка

1. **Загрузите хост** как в [Solo Developer](./solo-developer.md).
2. **Замените `hermes.service`** на шаблонный юнит (`hermes@.service`):

```ini
[Unit]
Description=Hermes Agent for %i
After=network-online.target

[Service]
Type=simple
User=%i
WorkingDirectory=/home/%i
ExecStart=/usr/local/bin/hermes run
EnvironmentFile=-/home/%i/.hermes/.env
# ... all the hardening bits from templates/systemd/hermes.service

[Install]
WantedBy=multi-user.target
```

Затем:

```bash
# For each dev or client:
adduser --disabled-password --gecos "" alice
sudo -u alice curl -sSL https://install.hermes.nous.ai | bash
cp templates/config/production.yaml /home/alice/.hermes/config.yaml
chown alice:alice /home/alice/.hermes/config.yaml
systemctl enable --now hermes@alice.service
```

3. **Централизуйте Langfuse** как в [Solo Developer](./solo-developer.md#install), затем каждый `config.yaml` указывает `telemetry.langfuse.host` на тот же внутренний URL. Каждый профиль работает в своём проекте Langfuse для изоляции.

## Разделение по клиентам

- **`profile:`** в конфиге Hermes — `quarantine` (недоверенный ввод для публичного бота) vs `trusted` (административный DM разработчика)
- **Каналы одобрения** — DM разработчика — единственный доверенный источник одобрения; каналы поддержки клиентов *никогда* не считаются доверенными
- **Директории LightRAG** — `~/.hermes/lightrag-<client>/` для каждого клиента; никогда не смешиваются
- **MCP** — PAT только на чтение для каждого клиента (`GITHUB_PAT_CLIENT_A`, `GITHUB_PAT_CLIENT_B`)
- **Журнал аудита** — append-only JSONL на сессию (session), централизованно в один append-only бакет, который разработчик может *читать*, но не *удалять* (упрощает проверки со стороны клиента)

## Маршрутизация стоимости в масштабе агентства

Используйте [`templates/config/production.yaml`](../../templates/config/production.yaml) как основу. Ключевые правила:

- **Триаж** (большая часть трафика): Cerebras Llama 70B — почти бесплатный тариф
- **Обычное программирование:** Kimi/Moonshot (дешёвый компетентный кодер)
- **«Сложное» программирование / архитектура:** Anthropic Sonnet — явный opt-in
- **Длинноконтекстные исследования:** Gemini 2.5 Pro
- **Глубокие рассуждения:** OpenAI reasoning model (opt-in)

С еженедельным `cost-report` → Discord ops channel аномалии стоимости всплывают до получения счёта.

## Compliance-дружественные настройки по умолчанию

- `memory_write_redaction: true` (не записывать секреты в LightRAG)
- `log_redaction: true`
- `security.webhook.max_body_bytes: 524288`
- `security.approval.approval_timeout: 120` — ни одно действие не зависает в очереди навсегда
- Еженощное резервное копирование с шифрованием per-client age-ключами

## Когда переходить на следующий уровень

- После ~20 разработчиков → переходите на полноценную Kubernetes-конфигурацию с подами на профиль, отдельными экземплярами Langfuse на клиента
- Регулируемые отрасли → хостите LLM тоже самостоятельно (vLLM или Ollama на GPU-сервере)
